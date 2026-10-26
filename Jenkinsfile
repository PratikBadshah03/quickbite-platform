pipeline {
    agent any
    environment {
        IMAGE_NAME = "pratikbadshah03/quickbite"
        APP_SERVER = "16.59.52.14"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PratikBadshah03/quickbite-platform.git'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }
        stage('Tag') {
            steps {
                sh 'docker tag ${IMAGE_NAME}:latest ${IMAGE_NAME}:${BUILD_NUMBER}'
            }
        }
        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }
        stage('Deploy') {
            steps {
                sshagent(['app-server-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} "mkdir -p ~/quickbite-platform"
                        scp -o StrictHostKeyChecking=no docker-compose.yml ubuntu@${APP_SERVER}:~/quickbite-platform/
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER} "cd ~/quickbite-platform && docker compose pull frontend && docker compose up -d"
                    '''
                }
            }
        }
    }
}
