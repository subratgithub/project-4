pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "009160029390.dkr.ecr.ap-south-1.amazonaws.com/testrepo"
        IMAGE_TAG = "latest"
        IMAGE_NAME = "${ECR_REPO}:${IMAGE_TAG}"
        REMOTE_HOST = "ec2-user@35.154.15.34"
        REMOTE_APP_NAME = "httpd-app"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/chinabudhi123/project-4.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME ."
            }
        }

        stage('Login to ECR') {
            steps {
                sh """
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REPO
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push $IMAGE_NAME"
            }
        }

        stage('Manual Approval') {
            steps {
                input message: "Approve deployment to EC2 at 35.154.15.34?", ok: "Deploy"
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ssh-access-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $REMOTE_HOST '
                            aws ecr get-login-password --region $AWS_REGION | \
                            docker login --username AWS --password-stdin $ECR_REPO &&
                            docker pull $IMAGE_NAME &&
                            docker stop $REMOTE_APP_NAME || true &&
                            docker rm $REMOTE_APP_NAME || true &&
                            docker run -d --name $REMOTE_APP_NAME -p 80:80 $IMAGE_NAME
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}

