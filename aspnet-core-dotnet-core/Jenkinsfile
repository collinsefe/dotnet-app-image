pipeline {
    agent any

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    // Define the environment and port based on the branch name
                    env.ENVIRONMENT = (env.BRANCH_NAME == 'master') ? 'prod' : 'dev'
                    env.PORT = (env.BRANCH_NAME == 'master') ? '8082' : '8081' 
                }
            }
        }

        stage('Build Application') {
            steps {
                echo "Building the application for the ${env.ENVIRONMENT} environment..."
                sh 'mvn clean install'
            }
        }

        stage('Run Application') {
            steps {
                echo "Running the Spring Boot application on port ${env.PORT} in ${env.ENVIRONMENT} environment..."
                sh """
                nohup java -jar target/*.jar --server.port=${env.PORT} > application.log 2>&1 &
                echo "Application started in detached mode with PID: \$!"
                """
            }
        }

        stage('Test Application') {
            steps {
                echo "Testing the API in the ${env.ENVIRONMENT} environment..."
                sh """
                curl -v -u greg:turnquist http://localhost:${env.PORT}/api/employees/3
                """
            }                       
        }
    }

    post {
        always {
            echo "Jenkins pipeline completed. The application is still running."
        }
    }
}
