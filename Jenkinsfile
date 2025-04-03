pipeline {
    agent any
    environment {
        AWS_REGION = "eu-north-1"
        ECR_REPO = "${env.AWS_ACCOUNT_ID}.dkr.ecr.us-east-1.amazonaws.com/myecr-repo"
        IMAGE_TAG = "latest"
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/khushig222/aws-project.git'
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                docker build -t my-web-app .
                docker tag my-web-app:latest $ECR_REPO:$IMAGE_TAG
                docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
        stage('Deploy Locally on EC2') {
            steps {
                sh '''
                # Authenticate Docker with AWS ECR
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO

                # Pull the latest Docker image
                docker pull $ECR_REPO:$IMAGE_TAG

                # Stop and remove old container if running
                docker stop web-container || true
                docker rm web-container || true

                # Run new container
                docker run -d --name web-container -p 80:3000 --env-file .env $ECR_REPO:$IMAGE_TAG
                '''
            }
        }
    }
}
