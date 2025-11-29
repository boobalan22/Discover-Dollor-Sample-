pipeline {
    agent any
    stages {
        stage('Build Backend & Docker Image') {
            steps {
                dir('backend') {
                    sh """
                        docker rmi -f shrsyc/dd_mean_backend || true
                        docker build -t shrsyc/dd_mean_backend .
                    """
                }
            }
        }
        stage('Build Frontend & Docker Image') {
            steps {
                dir('frontend') {
                    sh """
                        docker rmi -f shrsyc/dd_mean_frontend || true
                        docker build -t shrsyc/dd_mean_frontend .
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
                        docker push $DOCKER_USER/dd_mean_frontend
                        cd ..

                        
                        cd backend
                        docker push $DOCKER_USER/dd_mean_backend
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
