pipeline {
    agent any
    stages {
        stage('Build Backend & Docker Image') {
            steps {
                dir('backend') {
                    sh """
                        docker rmi -f boobu/backend-mean || true
                        docker build -t boobu/backend-mean .
                    """
                }
            }
        }
        stage('Build Frontend & Docker Image') {
            steps {
                dir('frontend') {
                    sh """
                        docker rmi -f boobu/frontend-mean || true
                        docker build -t boobu/frontend-mean .
                    """
                }
            }
        }
        stage("Push both Docker images to Docker Hub") { 
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        cd frontend
                        docker push $DOCKER_USER/frontend-mean
                        cd ..

                        
                        cd backend
                        docker push $DOCKER_USER/backend-mean
                        cd ..

                        docker logout
                    """
                }
            }
        }
        stage('Run Docker Compose') {
            steps {
                    sh '''
                        docker-compose down || true
                        docker-compose up -d
                    '''
            }
        }

    }
}
