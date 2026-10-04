pipeline {

    agent any

    environment {
        IMAGE_NAME = 'nginx-cicd-project'
        CONTAINER_NAME = 'nginx-cicd-container'
    }

    stages {

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
                        wget -q --spider http://127.0.0.1/

                    echo "Nginx image test successful!"

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

                    sleep 3

                    docker ps --filter name=${CONTAINER_NAME}
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                echo 'Checking deployed website...'

                sh '''
                    curl -f http://host.docker.internal:8081

                    echo ""
                    echo "Nginx deployment smoke test successful!"
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'CI/CD Pipeline completed successfully!'
            echo '======================================'
            echo 'Application: Nginx'
            echo 'Deployment Port: 8081'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'CI/CD Pipeline FAILED!'
            echo '======================================'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
