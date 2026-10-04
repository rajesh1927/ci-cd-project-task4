# End-to-End CI/CD Pipeline Report

## 1. Project Title

**End-to-End CI/CD Pipeline for a Static Web Application using Jenkins, Docker and Nginx**

---

## 2. Objective

The objective of this project is to design and implement an end-to-end CI/CD pipeline for a simple static web application.

The pipeline automatically:

- Retrieves source code from GitHub
- Builds a Docker image
- Tests the Nginx application
- Deploys the application using Docker
- Performs a smoke test after deployment

The web application is served using the Nginx web server.

---

## 3. Technologies and Tools Used

| Tool | Purpose |
|---|---|
| Git | Version control |
| GitHub | Source code repository |
| Jenkins | CI/CD automation |
| Docker | Containerization and deployment |
| Nginx | Web server |
| HTML | Static web application |
| cURL | Deployment smoke testing |
| Wget | Container-level testing |

---

## 4. Application Architecture

```text
Developer
    |
    v
GitHub Repository
    |
    v
Jenkins
    |
    +----------------+
    |                |
    v                v
  Build             Test
    |                |
    +-------+--------+
            |
            v
       Docker Image
            |
            v
         Deploy
            |
            v
      Nginx Container
            |
            v
       Smoke Test
            |
            v
     Static Web Page
```

---

## 5. CI/CD Pipeline Stages

### Stage 1: Source Code Checkout

**Tool:** Git / GitHub / Jenkins SCM

Jenkins retrieves the latest source code from the GitHub repository.

Repository:

```text
https://github.com/rajesh1927/ci-cd-project-task4
```

The Jenkins Pipeline job is configured with:

```text
Pipeline script from SCM
SCM: Git
Branch: main
Script Path: Jenkinsfile
```

Jenkins automatically checks out the source code before executing the pipeline.

**Purpose:**  
To ensure that the pipeline always works with the latest version of the application source code.

---

### Stage 2: Build

**Tools:** Docker, Dockerfile

Jenkins builds the Docker image using the project Dockerfile.

Command:

```bash
docker build -t nginx-cicd-project:latest .
```

The Dockerfile uses the lightweight Nginx Alpine image:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
```

**Purpose:**  
To create a consistent and portable Docker image containing the web application and Nginx web server.

---

### Stage 3: Test

**Tools:** Docker, Wget

A temporary Docker container is started from the newly created image.

The pipeline checks whether Nginx is responding:

```bash
docker exec test-nginx-container \
    wget -q --spider http://127.0.0.1/
```

If the test succeeds, the temporary container is removed.

If the test fails, the pipeline stops and deployment is not performed.

**Purpose:**  
To verify that the Docker image contains a working Nginx web server before deploying it.

---

### Stage 4: Deployment

**Tools:** Docker, Nginx

After successful testing, Jenkins removes the previous application container and starts a new one.

```bash
docker rm -f nginx-cicd-container || true

docker run -d \
    --name nginx-cicd-container \
    -p 8081:80 \
    nginx-cicd-project:latest
```

Nginx listens on port `80` inside the container.

Docker maps port `80` to port `8081` on the host.

```text
Host Port 8081
      |
      v
Container Port 80
      |
      v
Nginx
```

The application can be accessed at:

```text
http://localhost:8081
```

**Purpose:**  
To automatically deploy the tested Docker image.

---

### Stage 5: Smoke Test

**Tools:** cURL

After deployment, Jenkins verifies that the deployed application is accessible.

Because Jenkins is running inside Docker on Docker Desktop, the host application is accessed using:

```bash
curl -f http://host.docker.internal:8081
```

If the HTTP request succeeds, the deployment is considered successful.

**Purpose:**  
To verify that the deployed application is actually responding after deployment.

---

## 6. Pipeline Configuration

The complete Jenkins pipeline is defined in the `Jenkinsfile`.

```groovy
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
```

---

## 7. Pipeline Execution Flow

The final CI/CD workflow is:

```text
GitHub
   |
   v
Jenkins Checkout
   |
   v
Build Docker Image
   |
   v
Test Nginx Container
   |
   v
Deploy Nginx Container
   |
   v
Smoke Test
   |
   v
SUCCESS
```

---

## 8. Expected Result

A successful Jenkins execution should show:

```text
Build          SUCCESS
Test           SUCCESS
Deploy         SUCCESS
Smoke Test     SUCCESS
```

Final result:

```text
Finished: SUCCESS
```

The deployed website is available at:

```text
http://localhost:8081
```

---

## 9. Benefits of the Pipeline

This CI/CD pipeline provides:

- Automated build process
- Automated testing
- Automated deployment
- Consistent Docker-based environment
- Nginx web server deployment
- Deployment verification
- Reduced manual deployment effort
- Repeatable deployment process

---

## 10. Conclusion

The project successfully demonstrates an end-to-end CI/CD workflow using GitHub, Jenkins, Docker and Nginx.

The pipeline automatically builds, tests, deploys and verifies a static web application.

The final workflow is:

```text
GitHub → Jenkins → Build → Test → Deploy → Smoke Test → Nginx
```

This satisfies the requirements for an end-to-end CI/CD pipeline with documented stages, tools, pipeline configuration and deployment verification.