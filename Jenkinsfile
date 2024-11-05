pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'collinsefe-dockerhub' 
        DOCKER_IMAGE = 'collinsefe/dotnet-app-image'
        DOCKER_TAG = 'latest'
        PORT = 8084
        APP_DIR = 'aspnet-core-dotnet-core' 
    }

    stages {
        stage('Clean-up Docker') {
            steps {
                echo "Moving to the application directory: ${APP_DIR}..."
                dir(APP_DIR) {
                echo 'Current directory:'
                sh 'pwd' 
                echo 'Removing existing containers and images...'
                sh 'sudo docker rm -f $(sudo docker ps -a -q) || true'
                sh 'sudo docker image rm -f $(sudo docker images -a -q) || true'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'sudo docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Logging in to Docker Hub and pushing the image...'
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo "$DOCKER_PASSWORD" | sudo docker login -u "$DOCKER_USERNAME" --password-stdin'
                }
                sh 'sudo docker push ${DOCKER_IMAGE}:${DOCKER_TAG}'
            }
        }

        stage('Run Docker Container') {
            steps {
                echo "Running the Docker container on port ${PORT}..."
                sh "sudo docker run -d -p ${PORT}:80 ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. The Docker container is running.'
            sh 'sudo docker ps -a' 
        }
    }
}
