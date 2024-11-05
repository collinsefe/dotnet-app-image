pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'collinsefe-dockerhub'
        DOCKER_IMAGE = 'collinsefe/dotnet-app-image'
        DOCKER_TAG = 'latest'
        PORT = 8084
        APP_DIR = 'aspnet-core-dotnet-core'
        AWS_CREDENTIALS = 'aws-credentials' 
        ECS_CLUSTER_NAME = 'demo-app-cluster' 
        ECS_SERVICE_NAME = 'collins-cap-demo-app' 
        ECS_TASK_DEFINITION = 'demo-app-task' 
    }

    stages {
        stage('Clean-up Docker') {
            steps {
                dir(APP_DIR) {
                    echo 'Moving to APP directory...'
                    sh 'pwd'
                    sh 'ls -la'
                    sh 'sudo docker rm -f $(sudo docker ps -a -q) || true'
                    sh 'sudo docker image rm -f $(sudo docker images -a -q) || true'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir(APP_DIR) {
                    echo 'Building Docker image...'
                    sh 'sudo docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . || exit 1'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Logging in to Docker Hub and pushing the image...'
                dir(APP_DIR) {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo "$DOCKER_PASSWORD" | sudo docker login -u "$DOCKER_USERNAME" --password-stdin || exit 1'
                    }
                    sh 'sudo docker push ${DOCKER_IMAGE}:${DOCKER_TAG} || exit 1'
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                        echo 'Deploying to ECS...'

                        sh 'aws --version || exit 1'
                        
                        sh """
                        aws ecs register-task-definition \
                          --family ${ECS_TASK_DEFINITION} \
                          --container-definitions '[{
                            "name": "container",
                            "image": "${DOCKER_IMAGE}:${DOCKER_TAG}",
                            "essential": true,
                            "memory": 512,
                            "cpu": 256,
                            "portMappings": [{
                              "containerPort": 80,
                              "hostPort": 8084
                            }]
                          }]' || exit 1
                        """

                        // Update ECS service
                        sh """
                        aws ecs update-service \
                          --cluster ${ECS_CLUSTER_NAME} \
                          --service ${ECS_SERVICE_NAME} \
                          --force-new-deployment || exit 1
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Checking Docker containers and ECS services...'
            sh 'sudo docker ps -a || true'
            sh 'aws ecs list-services --cluster ${ECS_CLUSTER_NAME} || true'
        }
    }
}
