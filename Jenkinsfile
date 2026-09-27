pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '627917841032.dkr.ecr.ap-south-1.amazonaws.com'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    echo "Checking Jenkins build environment..."
                    uname -a
                    docker --version
                    aws --version
                '''
            }
        }

        stage('AWS ECR Login') {
            steps {
                sh '''
                    aws ecr get-login-password --region "$AWS_REGION" | \
                    docker login --username AWS --password-stdin "$ECR_REGISTRY"
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build -t "$ECR_REGISTRY/streaming-auth:1.0.0" \
                      -f backend/authService/Dockerfile backend

                    docker build -t "$ECR_REGISTRY/streaming-service:1.0.1" \
                      -f backend/streamingService/Dockerfile backend

                    docker build -t "$ECR_REGISTRY/streaming-admin:1.0.0" \
                      -f backend/adminService/Dockerfile backend

                    docker build -t "$ECR_REGISTRY/streaming-chat:1.0.0" \
                      -f backend/chatService/Dockerfile backend

                    docker build \
                      --build-arg REACT_APP_AUTH_API_URL=/api \
                      --build-arg REACT_APP_STREAMING_API_URL=/api/streaming \
                      --build-arg REACT_APP_STREAMING_PUBLIC_URL=/ \
                      --build-arg REACT_APP_ADMIN_API_URL=/api/admin \
                      --build-arg REACT_APP_CHAT_API_URL=/api/chat \
                      --build-arg REACT_APP_CHAT_SOCKET_URL=/socket.io \
                      -t "$ECR_REGISTRY/streaming-frontend:1.0.3" \
                      frontend
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    docker push "$ECR_REGISTRY/streaming-auth:1.0.0"
                    docker push "$ECR_REGISTRY/streaming-service:1.0.1"
                    docker push "$ECR_REGISTRY/streaming-admin:1.0.0"
                    docker push "$ECR_REGISTRY/streaming-chat:1.0.0"
                    docker push "$ECR_REGISTRY/streaming-frontend:1.0.3"
                '''
            }
        }
    }

    post {
        success {
            echo 'All five StreamingApp images were successfully built and pushed to Amazon ECR.'
        }
        failure {
            echo 'Jenkins pipeline failed. Check the failed stage and console output.'
        }
    }
}
