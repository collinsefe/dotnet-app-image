pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = 'aws-credentials' 
        AWS_ACCOUNT_ID = '684361860346' 
        AWS_REGION = 'eu-west-2'
        ECR_REPO_NAME = 'cap-demo-app'
        DOCKER_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
        DOCKER_TAG = 'latest'
        PORT = 8084
        APP_DIR = 'aspnet-core-dotnet-core'
    }

    stages {
        stage('Clean-up Docker') {
            steps {
                dir(APP_DIR) {
                    echo 'Moving to APP directory...'
                    sh 'pwd'
                    sh 'ls -la'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir(APP_DIR) {
                    echo 'Building Docker image...'
                    sh "sudo docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . || exit 1"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                echo 'Logging in to Amazon ECR and pushing the image...'
                dir(APP_DIR) {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                        // Login to ECR
                        sh "aws ecr get-login-password --region ${AWS_REGION} | sudo docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    }
                    // Push the Docker image to ECR
                    sh "sudo docker push ${DOCKER_IMAGE}:${DOCKER_TAG} || exit 1"
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                        echo 'Deploying to ECS...'

                        // Register ECS task definition
                        sh """
                        aws ecs register-task-definition \
                          --family ${ECS_REPO_NAME} \
                          --container-definitions '[{
                            "name": "container",
                            "image": "${DOCKER_IMAGE}:${DOCKER_TAG}",
                            "essential": true,
                            "memory": 512,
                            "cpu": 256,
                            "portMappings": [{
                              "containerPort": 80,
                              "hostPort": ${PORT}
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
        
        stage('Test Application') {
            steps {
                echo "Testing if the application is running on ${APP_ENDPOINT}..."

                sleep(20)

                script {
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' ${APP_ENDPOINT}", returnStdout: true).trim()
                    if (response == '200') {
                        echo 'Application is running and responded with HTTP 200 OK!'
                    } else {
                        error "Application test failed! Endpoint responded with HTTP ${response}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Checking Docker containers and ECS services...'
             withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
            sh 'aws ecs list-services --cluster ${ECS_CLUSTER_NAME} || true'
             }
        }
    }
}
