pipeline {
    agent any
    stages {
        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh """
                        docker rmi -f backend-mean || true
                        docker build -t backend-mean .
                    """
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh """
                        docker rmi -f frontend-mean || true
                        docker build -t frontend-mean .
                    """
                }
            }
        }

        stage("Tag & Push Images to Docker Hub") { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        docker tag frontend-mean $DOCKER_USER/frontend-mean:latest
                        docker tag backend-mean $DOCKER_USER/backend-mean:latest

                        docker push $DOCKER_USER/frontend-mean:latest
                        docker push $DOCKER_USER/backend-mean:latest

                        docker logout
                    """
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                sh '''
                    docker-compose down || true
                    docker-compose up -d --build
                '''
            }
        }

        stage('Clean Dangling Images') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }
}
