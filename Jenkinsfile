pipeline {

    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    environment {
        AWS_REGION  = 'ap-south-1'
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
                    set -e

                    echo "======================================"
                    echo "Checking Jenkins build environment"
                    echo "======================================"

                    uname -a
                    docker --version
                    aws --version
                '''
            }
        }

        stage('Verify AWS Credentials') {
            steps {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'streamingapp-aws-final'
                    ]
                ]) {
                    sh '''
                        set -e

                        echo "======================================"
                        echo "Verifying AWS identity"
                        echo "======================================"

                        aws sts get-caller-identity \
                            --region "$AWS_REGION"

                        echo "AWS credentials are working."
                    '''
                }
            }
        }

        stage('AWS ECR Login') {
            steps {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'streamingapp-aws-final'
                    ]
                ]) {
                    sh '''
                        set -e

                        echo "======================================"
                        echo "Logging in to Amazon ECR"
                        echo "======================================"

                        aws ecr get-login-password \
                            --region "$AWS_REGION" | \
                            docker login \
                            --username AWS \
                            --password-stdin "$ECR_REGISTRY"

                        echo "ECR login successful."
                    '''
                }
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    set -e

                    echo "======================================"
                    echo "Building StreamingApp Docker Images"
                    echo "======================================"

                    echo "Building Auth Service..."
                    docker build \
                        -t "$ECR_REGISTRY/streaming-auth:1.0.0" \
                        -f backend/authService/Dockerfile \
                        backend

                    echo "Building Streaming Service..."
                    docker build \
                        -t "$ECR_REGISTRY/streaming-service:1.0.1" \
                        -f backend/streamingService/Dockerfile \
                        backend

                    echo "Building Admin Service..."
                    docker build \
                        -t "$ECR_REGISTRY/streaming-admin:1.0.0" \
                        -f backend/adminService/Dockerfile \
                        backend

                    echo "Building Chat Service..."
                    docker build \
                        -t "$ECR_REGISTRY/streaming-chat:1.0.0" \
                        -f backend/chatService/Dockerfile \
                        backend

                    echo "Building Frontend..."
                    docker build \
                        --build-arg REACT_APP_AUTH_API_URL=/api \
                        --build-arg REACT_APP_STREAMING_API_URL=/api/streaming \
                        --build-arg REACT_APP_STREAMING_PUBLIC_URL=/ \
                        --build-arg REACT_APP_ADMIN_API_URL=/api/admin \
                        --build-arg REACT_APP_CHAT_API_URL=/api/chat \
                        --build-arg REACT_APP_CHAT_SOCKET_URL=/socket.io \
                        -t "$ECR_REGISTRY/streaming-frontend:1.0.3" \
                        frontend

                    echo "======================================"
                    echo "All 5 images built successfully."
                    echo "======================================"

                    docker images | grep "$ECR_REGISTRY"
                '''
            }
        }

        stage('Push Images to ECR') {
            steps {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'streamingapp-aws-final'
                    ]
                ]) {
                    sh '''
                        set -e

                        echo "======================================"
                        echo "Pushing Images to Amazon ECR"
                        echo "======================================"

                        echo "Pushing Auth Service..."
                        docker push "$ECR_REGISTRY/streaming-auth:1.0.0"

                        echo "Pushing Streaming Service..."
                        docker push "$ECR_REGISTRY/streaming-service:1.0.1"

                        echo "Pushing Admin Service..."
                        docker push "$ECR_REGISTRY/streaming-admin:1.0.0"

                        echo "Pushing Chat Service..."
                        docker push "$ECR_REGISTRY/streaming-chat:1.0.0"

                        echo "Pushing Frontend..."
                        docker push "$ECR_REGISTRY/streaming-frontend:1.0.3"

                        echo "======================================"
                        echo "All 5 images pushed successfully."
                        echo "======================================"
                    '''
                }
            }
        }

        stage('Verify ECR Images') {
            steps {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'streamingapp-aws-final'
                    ]
                ]) {
                    sh '''
                        set -e

                        echo "======================================"
                        echo "Verifying Images in Amazon ECR"
                        echo "======================================"

                        echo "Auth Service:"
                        aws ecr describe-images \
                            --repository-name streaming-auth \
                            --image-ids imageTag=1.0.0 \
                            --region "$AWS_REGION" \
                            --query 'imageDetails[0].{Tag:imageTags[0],Digest:imageDigest}' \
                            --output table

                        echo "Streaming Service:"
                        aws ecr describe-images \
                            --repository-name streaming-service \
                            --image-ids imageTag=1.0.1 \
                            --region "$AWS_REGION" \
                            --query 'imageDetails[0].{Tag:imageTags[0],Digest:imageDigest}' \
                            --output table

                        echo "Admin Service:"
                        aws ecr describe-images \
                            --repository-name streaming-admin \
                            --image-ids imageTag=1.0.0 \
                            --region "$AWS_REGION" \
                            --query 'imageDetails[0].{Tag:imageTags[0],Digest:imageDigest}' \
                            --output table

                        echo "Chat Service:"
                        aws ecr describe-images \
                            --repository-name streaming-chat \
                            --image-ids imageTag=1.0.0 \
                            --region "$AWS_REGION" \
                            --query 'imageDetails[0].{Tag:imageTags[0],Digest:imageDigest}' \
                            --output table

                        echo "Frontend:"
                        aws ecr describe-images \
                            --repository-name streaming-frontend \
                            --image-ids imageTag=1.0.3 \
                            --region "$AWS_REGION" \
                            --query 'imageDetails[0].{Tag:imageTags[0],Digest:imageDigest}' \
                            --output table

                        echo "======================================"
                        echo "ECR verification successful."
                        echo "======================================"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '''
========================================
STREAMINGAPP JENKINS PIPELINE SUCCESS
========================================
All five Docker images were:
1. Built successfully
2. Pushed to Amazon ECR
3. Verified in Amazon ECR
========================================
'''
        }

        failure {
            echo '''
========================================
STREAMINGAPP JENKINS PIPELINE FAILED
========================================
Check the failed stage and console output.
========================================
'''
        }

        always {
            sh '''
                echo "Jenkins pipeline completed."
            '''
        }
    }
}
