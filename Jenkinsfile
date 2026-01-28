pipeline {
    agent any

    environment {
        IMAGE_NAME = "aiops-nginx-site"
        IMAGE_TAG  = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh '''
                  docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                echo "Running Trivy scan..."
                sh '''
                  docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image $IMAGE_NAME:$IMAGE_TAG || true
                '''
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo "Running SonarQube scan..."
                sh '''
                  docker run --rm \
                    -e SONAR_HOST_URL=http://192.168.56.50:9000 \
                    -v $(pwd):/usr/src \
                    sonarsource/sonar-scanner-cli
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying container..."
                sh '''
                  docker rm -f aiops-nginx-prod || true
                  docker run -d -p 8086:80 --name aiops-nginx-prod $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS - notifying n8n"
            sh '''
              curl -X POST http://192.168.56.50:5678/webhook/jenkins-success \
                -H "Content-Type: application/json" \
                -d '{"job":"aiseclab-pipeline","status":"SUCCESS"}'
            '''
        }

        failure {
            echo "FAILED - notifying n8n"
            sh '''
              curl -X POST http://192.168.56.50:5678/webhook/jenkins-failure \
                -H "Content-Type: application/json" \
                -d '{"job":"aiseclab-pipeline","status":"FAILURE"}'
            '''
        }
    }
}
