
pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1'
        AWS_REGION = 'ap-south-1'

        AWS_ACCOUNT_ID = '627917841032'
        ECR_REGISTRY = '627917841032.dkr.ecr.ap-south-1.amazonaws.com'

        AWS_CREDENTIALS_ID = 'streamingapp-aws-final'

        AUTH_IMAGE = 'streaming-auth'
        STREAMING_IMAGE = 'streaming-service'
        ADMIN_IMAGE = 'streaming-admin'
        CHAT_IMAGE = 'streaming-chat'
        FRONTEND_IMAGE = 'streaming-frontend'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                echo 'Checking out StreamingApp source code from GitHub'
                checkout scm
                sh 'git log -1 --oneline'
            }
        }

        stage('Verify Tools') {
            steps {
                echo 'Verifying Docker and AWS CLI installation'

                sh '''
                    docker --version
                    aws --version
                '''
            }
        }

        stage('Verify AWS Credentials') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: "${AWS_CREDENTIALS_ID}"]
                ]) {
                    sh '''
                        echo "Checking AWS identity..."
                        aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('AWS ECR Login') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: "${AWS_CREDENTIALS_ID}"]
                ]) {
                    sh '''
                        echo "Logging in to Amazon ECR..."

                        aws ecr get-login-password \
                            --region ${AWS_DEFAULT_REGION} \
                        | docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Build Images') {
            steps {
                echo 'Building all StreamingApp Docker images'

                sh '''
                    set -e

                    echo "Building Auth Service..."
                    docker build \
                        -t ${ECR_REGISTRY}/${AUTH_IMAGE}:1.0.0 \
                        -f backend/authService/Dockerfile \
                        backend/authService

                    echo "Building Streaming Service..."
                    docker build \
                        -t ${ECR_REGISTRY}/${STREAMING_IMAGE}:1.0.1 \
                        -f backend/streamingService/Dockerfile \
                        backend/streamingService

                    echo "Building Admin Service..."
                    docker build \
                        -t ${ECR_REGISTRY}/${ADMIN_IMAGE}:1.0.0 \
                        -f backend/adminService/Dockerfile \
                        backend/adminService

                    echo "Building Chat Service..."
                    docker build \
                        -t ${ECR_REGISTRY}/${CHAT_IMAGE}:1.0.0 \
                        -f backend/chatService/Dockerfile \
                        backend/chatService

                    echo "Building Frontend..."
                    docker build \
                        -t ${ECR_REGISTRY}/${FRONTEND_IMAGE}:1.0.3 \
                        --build-arg REACT_APP_AUTH_API_URL=/api \
                        --build-arg REACT_APP_STREAMING_API_URL=/api/streaming \
                        --build-arg REACT_APP_STREAMING_PUBLIC_URL=/ \
                        --build-arg REACT_APP_ADMIN_API_URL=/api/admin \
                        --build-arg REACT_APP_CHAT_API_URL=/api/chat \
                        --build-arg REACT_APP_CHAT_SOCKET_URL=/socket.io \
                        -f frontend/Dockerfile \
                        frontend

                    echo "All Docker images built successfully."
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                echo 'Pushing Docker images to Amazon ECR'

                sh '''
                    set -e

                    docker push ${ECR_REGISTRY}/${AUTH_IMAGE}:1.0.0
                    docker push ${ECR_REGISTRY}/${STREAMING_IMAGE}:1.0.1
                    docker push ${ECR_REGISTRY}/${ADMIN_IMAGE}:1.0.0
                    docker push ${ECR_REGISTRY}/${CHAT_IMAGE}:1.0.0
                    docker push ${ECR_REGISTRY}/${FRONTEND_IMAGE}:1.0.3

                    echo "All Docker images pushed successfully."
                '''
            }
        }

        stage('Verify ECR Images') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: "${AWS_CREDENTIALS_ID}"]
                ]) {
                    sh '''
                        set -e

                        echo "Verifying Auth image..."
                        aws ecr describe-images \
                            --repository-name ${AUTH_IMAGE} \
                            --image-ids imageTag=1.0.0 \
                            --region ${AWS_DEFAULT_REGION}

                        echo "Verifying Streaming image..."
                        aws ecr describe-images \
                            --repository-name ${STREAMING_IMAGE} \
                            --image-ids imageTag=1.0.1 \
                            --region ${AWS_DEFAULT_REGION}

                        echo "Verifying Admin image..."
                        aws ecr describe-images \
                            --repository-name ${ADMIN_IMAGE} \
                            --image-ids imageTag=1.0.0 \
                            --region ${AWS_DEFAULT_REGION}

                        echo "Verifying Chat image..."
                        aws ecr describe-images \
                            --repository-name ${CHAT_IMAGE} \
                            --image-ids imageTag=1.0.0 \
                            --region ${AWS_DEFAULT_REGION}

                        echo "Verifying Frontend image..."
                        aws ecr describe-images \
                            --repository-name ${FRONTEND_IMAGE} \
                            --image-ids imageTag=1.0.3 \
                            --region ${AWS_DEFAULT_REGION}

                        echo "All ECR images verified successfully."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '''
            ==========================================
            SUCCESS: STREAMINGAPP CI PIPELINE PASSED
            ==========================================
            All Docker images were built, pushed to
            Amazon ECR, and verified successfully.
            '''
        }

        failure {
            echo '''
            ==========================================
            FAILURE: STREAMINGAPP CI PIPELINE FAILED
            ==========================================
            Check the Jenkins Console Output to identify
            the stage and error.
            '''
        }

        always {
            echo 'StreamingApp Jenkins pipeline execution completed.'
        }
    }
}