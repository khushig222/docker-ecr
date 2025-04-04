pipeline {
    agent any

    environment {
        AWS_REGION = "eu-north-1"
        AWS_ACCOUNT_ID = "${env.AWS_ACCOUNT_ID}"
        ECR_REPOSITORY = "myecr-repo"
        KUBE_CONFIG = credentials('kube-config') 
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/khushig222/aws-project.git'
            }
        }

        stage('AWS ECR Login') {
            steps {
                sh """
                aws ecr get-login-password --region $AWS_REGION | kubectl create secret docker-registry ecr-secret \
                    --docker-server=${env.AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com \
                    --docker-username=AWS \
                    --docker-password=$(aws ecr get-login-password --region $AWS_REGION) || true
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kube-config']) {
                    sh 'kubectl apply -f k8s-deployment.yml'
                    sh 'kubectl apply -f k8s-service.yml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get pods'
                sh 'kubectl get svc'
            }
        }
    }
}

