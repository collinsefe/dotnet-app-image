pipeline {
    agent any
    environment {
        DOCKER_HUB_USERNAME = 'collinsefe'
        IMAGE_NAME = "${DOCKER_HUB_USERNAME}/dotnet-app-image"
        TAG = 'latest'
    }
    stages {
        stage('Prepare') {
            steps {
                script {
                    dir('aspnet-core-dotnet-core') {
                        sh 'ls -la'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        echo "Building Docker image..."
                        sh "sudo docker build -t ${IMAGE_NAME}:${TAG} aspnet-core-dotnet-core"
                    } catch (Exception e) {
                        error 'Docker build failed. Exiting...'
                    }
                }
            }
        }

         stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'collinsefe-dockerhub', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    sh "echo ${DOCKER_HUB_PASSWORD} | sudo docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    try {
                        echo "Pushing Docker image to Docker Hub..."
                        sh "sudo docker push ${IMAGE_NAME}:${TAG}"
                    } catch (Exception e) {
                        error 'Docker push failed. Exiting...'
                    }
                }
            }
        }

        stage('Clean Up Local Docker Image') {
            steps {
                script {
                    echo "Cleaning up local Docker image..."
                    sh "sudo docker rmi ${IMAGE_NAME}:${TAG}"
                }
            }
        }
    }
    post {
        success {
            echo "Docker image has been pushed successfully!"
        }
    }
}
