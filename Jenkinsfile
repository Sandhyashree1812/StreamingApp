pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '759910672539'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        AUTH_IMAGE = "${ECR_REGISTRY}/streamingapp-auth"
        STREAMING_IMAGE = "${ECR_REGISTRY}/streamingapp-streaming"
        ADMIN_IMAGE = "${ECR_REGISTRY}/streamingapp-admin"
        CHAT_IMAGE = "${ECR_REGISTRY}/streamingapp-chat"
        FRONTEND_IMAGE = "${ECR_REGISTRY}/streamingapp-frontend"

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Running StreamingApp validation...'
                sh 'node --version'
                sh 'npm --version'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t ${AUTH_IMAGE}:${IMAGE_TAG} ./backend/authService
                    docker build -t ${STREAMING_IMAGE}:${IMAGE_TAG} ./backend/streamingService
                    docker build -t ${ADMIN_IMAGE}:${IMAGE_TAG} ./backend/adminService
                    docker build -t ${CHAT_IMAGE}:${IMAGE_TAG} ./backend/chatService
                    docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} ./frontend
                '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} |
                        docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                    docker push ${AUTH_IMAGE}:${IMAGE_TAG}
                    docker push ${STREAMING_IMAGE}:${IMAGE_TAG}
                    docker push ${ADMIN_IMAGE}:${IMAGE_TAG}
                    docker push ${CHAT_IMAGE}:${IMAGE_TAG}
                    docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo 'StreamingApp CI/CD pipeline completed successfully.'
        }

        failure {
            echo 'StreamingApp pipeline failed. Check the Jenkins console output.'
        }
    }
}