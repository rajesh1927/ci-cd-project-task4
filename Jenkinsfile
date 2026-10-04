pipeline {
    agent any

    environment {
        IMAGE_NAME = 'nginx-cicd-project'
        CONTAINER_NAME = 'nginx-cicd-container'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building Docker image...'

                sh '''
                    docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Test') {
		steps {
        echo 'Testing Docker image...'

        sh '''
            docker rm -f test-nginx-container || true

            docker run -d \
                --name test-nginx-container \
                ${IMAGE_NAME}:latest

            sleep 3

            docker exec test-nginx-container \
                wget -q --spider http://localhost/ || exit 1

            docker rm -f test-nginx-container
        '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying Nginx application...'

                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 8081:80 \
                        ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                echo 'Checking deployed website...'

                sh '''
                    sleep 3
                    curl -f http://localhost:8081
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
