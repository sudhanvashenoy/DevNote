pipeline {
    agent { label 'ec2-agent' }

    environment {
        PROD_USER = 'ubuntu'
        PROD_HOST = '13.203.217.222'
        IMAGE_NAME = 'static-app'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Pulling code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Deploy to Production') {
            steps {
                echo '🚀 Deploying to Production EC2...'
                sshagent(['prod-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no \
                        ${PROD_USER}@${PROD_HOST} "
                            docker stop static-app || true &&
                            docker rm static-app || true &&
                            docker run -d \
                                --name static-app \
                                -p 80:80 \
                                static-app:latest
                        "
                    '''
                }
            }
        }
    }

    post {
        success { echo '✅ Site is LIVE on Production!' }
        failure { echo '❌ Pipeline Failed!' }
    }
}
