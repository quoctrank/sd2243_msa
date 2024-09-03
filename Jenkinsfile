pipeline {
    agent none
    environment {
        REGION = "ap-southeast-1"
        ECR_URL = "7673xxx.dkr.ecr.${REGION}.amazonaws.com"
        ECR_FRONTEND = "frontend"
        ECR_BACKEND = "backend"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {
        stage('Build and Push Backend') {
            agent any
            steps {
                withAWS(region: 'ap-southeast-1', credentials: 'aws-credential') {
                    // Login to ECR
                    sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_URL}"

                    // Build Docker image
                    sh "docker build -t ${ECR_BACKEND}:${IMAGE_TAG} src/backend/"

                    // Tag Docker image
                    sh "docker tag ${ECR_BACKEND}:${IMAGE_TAG} ${ECR_URL}/${ECR_BACKEND}"

                    // Push Docker image to ECR
                    sh "docker push ${ECR_URL}/${ECR_BACKEND}"
                }
            }
        }
        stage('Build and Push Frontend') {
            agent any
            steps {
                withAWS(region: 'ap-southeast-1', credentials: 'aws-credential') {
                    // Login to ECR
                    sh "aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_URL}"

                    // Build Docker image
                    sh "docker build -t ${ECR_FRONTEND}:${IMAGE_TAG} src/frontend/"

                    // Tag Docker image
                    sh "docker tag ${ECR_FRONTEND}:${IMAGE_TAG} ${ECR_URL}/${ECR_FRONTEND}"

                    // Push Docker image to ECR
                    sh "docker push ${ECR_URL}/${ECR_FRONTEND}"
                }
            }
        }
        stage('Deploy to K8S') {
            agent any
            steps {
                withAWS(region: 'ap-southeast-1', credentials: 'aws-credential') {
                    sh "aws eks update-kubeconfig --name sd2243-aws-cluster"
                    sh "kubectl apply -f k8s/aws/mongodb.yaml"
                    sh "kubectl apply -f k8s/aws/backend.yaml"
                    sh "kubectl apply -f k8s/aws/frontend.yaml"
                }
            }
        }
    }
}
