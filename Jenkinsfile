pipeline {
    agent any

    environment {
        IMAGE_NAME = "aiops-nginx-site"
        IMAGE_TAG  = "latest"
        SONAR_HOST = "http://192.168.56.50:9000"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📥 Checking out source code..."
                checkout scm
            }
        }

        // ===================== SAST =====================
        stage('SAST - SonarQube Scan') {
            steps {
                echo "🔍 Running SonarQube SAST scan..."
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                    docker run --rm \
                      -e SONAR_HOST_URL=$SONAR_HOST \
                      -e SONAR_LOGIN=$SONAR_TOKEN \
                      -v $(pwd):/usr/src \
                      sonarsource/sonar-scanner-cli
                    '''
                }
            }
        }

        // ===================== SCA =====================
        stage('SCA - OWASP Dependency Check') {
            steps {
                echo "🛡️ Running OWASP Dependency Check..."
                sh '''
                docker run --rm \
                  -v $(pwd):/src \
                  owasp/dependency-check \
                  --scan /src \
                  --format HTML \
                  --out /src/owasp-report
                '''
            }
        }

        // ===================== BUILD =====================
        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh '''
                  docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        // ===================== CONTAINER SCAN =====================
        stage('Container Scan - Trivy') {
            steps {
                echo "🔒 Running Trivy container vulnerability scan..."
                sh '''
                docker run --rm \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  aquasec/trivy image $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        // ===================== DEPLOY =====================
        stage('Deploy') {
            steps {
                echo "🚀 Deploying application..."
                sh '''
                  docker rm -f aiops-nginx-prod || true
                  docker run -d -p 8086:80 --name aiops-nginx-prod $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline SUCCESS — notifying n8n"
            sh '''
              curl -X POST http://192.168.56.50:5678/webhook/jenkins-success \
                -H "Content-Type: application/json" \
                -d '{"job":"aiseclab-real-pipeline","status":"SUCCESS"}'
            '''
        }

        failure {
            echo "❌ Pipeline FAILED — notifying n8n"
            sh '''
              curl -X POST http://192.168.56.50:5678/webhook/jenkins-failure \
                -H "Content-Type: application/json" \
                -d '{"job":"aiseclab-real-pipeline","status":"FAILURE"}'
            '''
        }
    }
}
