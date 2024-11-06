pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = 'aws-credentials' 
        AWS_ACCOUNT_ID = '684361860346' 
        AWS_REGION = 'eu-west-2'
        ECR_REPO_NAME = 'cap-gem-app-test'
        DOCKER_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
        DOCKER_TAG = 'latest'
        APP_DIR = 'aspnet-core-dotnet-core'
        ECS_CLUSTER_NAME = 'app-cluster-test'
        ECS_SERVICE_NAME = 'app-service-test'
        ECS_TASK_DEFINITION = 'app-task-family-test' 
        APP_ENDPOINT = "app-alb-test-1968587331.eu-west-2.elb.amazonaws.com"
        CONTAINER_NAME = "app-task-test"
    }

    stages {
        stage('Clean-up Docker') {
            steps {
                dir(APP_DIR) {
                    echo 'Clean-up Docker Images...'
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
                    sh "sudo docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . || exit 1"
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                echo 'Logging in to Amazon ECR and pushing the image...'
                dir(APP_DIR) {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | sudo docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    }            
                    sh "sudo docker push ${DOCKER_IMAGE}:${DOCKER_TAG} || exit 1"
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                        echo 'Deploying to ECS...'
                        sh """
                        aws ecs register-task-definition \
                          --family ${ECS_TASK_DEFINITION} \
                          --container-definitions '[{
                            "name": "${CONTAINER_NAME}", 
                            "image": "${DOCKER_IMAGE}:${DOCKER_TAG}",
                            "essential": true,
                            "memory": 512,
                            "cpu": 256,
                            "portMappings": [{
                              "containerPort": 80,
                              "hostPort": 0
                            }]
                          }]' || exit 1
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
                    def response = sh(script: "curl -s -o /test/null -w '%{http_code}' ${APP_ENDPOINT}", returnStdout: true).trim()
                    if (response == '200') {
                        echo 'Application is running and responded with HTTP 200 OK!'
                    } else {
                        error "Application test failed! Endpoint responded with HTTP ${response}"
                    }
                }
            }
        }

        // stage('Verify Monitoring') {
        //     steps {
        //         script {
        //             echo 'Verifying CloudWatch Logs...'
        //             sh "aws logs describe-log-groups --log-group-name-prefix /ecs/${ECS_SERVICE_NAME}"

        //             echo 'Checking ECS Service Metrics...'
        //             sh """
        //                 aws cloudwatch get-metric-statistics --metric-name CPUUtilization \
        //                 --namespace AWS/ECS --period 60 --statistics Average \
        //                 --dimensions Name=ServiceName,Value=${ECS_SERVICE_NAME} Name=ClusterName,Value=${ECS_CLUSTER_NAME} \
        //                 --start-time \$(date -u -d '10 minutes ago' +%Y-%m-%dT%H:%M:%SZ) --end-time \$(date -u +%Y-%m-%dT%H:%M:%SZ)
        //             """
        //         }
        //     }
        // }
    }

    post {
        always {
            echo 'Pipeline completed. Checking ECS services...'
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                sh "aws ecs list-services --cluster ${ECS_CLUSTER_NAME} || true"
            }
        }
    }
}
