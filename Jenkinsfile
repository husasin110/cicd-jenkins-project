pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "myapp-repo"
        IMAGE_TAG = "${BUILD_NUMBER}"
        ACCOUNT_ID = "700125623035"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', 'https://github.com/husasin110/cicd-jenkins-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $ECR_REPO:$IMAGE_TAG ."
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-jenkins', region: AWS_REGION) {
                    sh """
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker tag $ECR_REPO:$IMAGE_TAG $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                docker push $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                """
            }
        }
    }
}

