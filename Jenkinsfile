pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "myapp-repo"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        ACCOUNT_ID = "700125623035"
        EC2_IP     = "3.88.113.13"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/husasin110/cicd-jenkins-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-jenkins', region: "${AWS_REGION}") {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker tag ${ECR_REPO}:${IMAGE_TAG} ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${EC2_IP} << EOF
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    
                    docker pull ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                    docker stop myapp || true
                    docker rm myapp || true
                    docker run -d -p 80:3000 --name myapp ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                    EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build & Deployment Successful!"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
        always {
            echo "Pipeline finished at: ${new Date()}"
        }
    }
}
