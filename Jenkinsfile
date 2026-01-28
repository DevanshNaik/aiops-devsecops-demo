pipeline {
    agent any

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
                sh 'docker build -t aiops-nginx-site:latest .'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying..."
                sh '''
                  docker rm -f aiops-nginx-prod || true
                  docker run -d -p 8086:80 --name aiops-nginx-prod aiops-nginx-site:latest
                '''
            }
        }
    }
}
