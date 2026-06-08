pipeline {
    agent { label 'ec2-agent' }

    environment {
        PROD_USER = 'ubuntu'
        PROD_HOST = '65.0.170.146'
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
                        # Save image as tar file
                        docker save static-app:latest | gzip > static-app.tar.gz

                        # Copy image to Production EC2
                        scp -o StrictHostKeyChecking=no \
                            static-app.tar.gz \
                            ubuntu@${PROD_HOST}:/home/ubuntu/

                        # SSH into Production and load + run image
                        ssh -o StrictHostKeyChecking=no ubuntu@${PROD_HOST} "
                            docker load < /home/ubuntu/static-app.tar.gz &&
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
