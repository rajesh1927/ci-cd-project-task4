# Nginx CI/CD Pipeline Project

An end-to-end CI/CD pipeline for deploying a simple static website using **GitHub, Jenkins, Docker, and Nginx**.

The project demonstrates automated source-code checkout, Docker image build, application testing, container deployment, and smoke testing.

---

## Project Objective

The objective of this project is to design and implement a simple end-to-end CI/CD pipeline with the following stages:

1. Build
2. Test
3. Deployment

The application is a simple static HTML website served using the Nginx web server.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Git | Version control |
| GitHub | Source code repository |
| Jenkins | CI/CD automation |
| Docker | Application containerization |
| Nginx | Web server |
| HTML | Static website |
| cURL / Wget | Application testing |

---

## Project Architecture

```text
                    Developer
                        |
                        v
                 +-------------+
                 |   GitHub    |
                 | Repository  |
                 +------+------+
                        |
                        | Source Code
                        v
                 +-------------+
                 |   Jenkins   |
                 |    CI/CD    |
                 +------+------+
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
       Build          Test         Deploy
          |             |             |
          +-------------+-------------+
                        |
                        v
                 +-------------+
                 |   Docker    |
                 |  Container  |
                 +------+------+
                        |
                        v
                 +-------------+
                 |    Nginx    |
                 | Web Server  |
                 +------+------+
                        |
                        v
                Static Website
                 Port 8081
```

---

## Project Structure

```text
ci-cd-project-task4/
│
├── index.html
├── nginx.conf
├── Dockerfile
├── Jenkinsfile
├── README.md
└── .dockerignore
```

---

# Application

The project contains a simple static HTML website.

The website is served by Nginx from:

```text
/usr/share/nginx/html
```

The default application port inside the Docker container is:

```text
80
```

The container port is mapped to:

```text
8081
```

Therefore, the application can be accessed using:

```text
http://localhost:8081
```

---

# Nginx Configuration

The `nginx.conf` file configures Nginx to listen on port `80` and serve the static website.

```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

# Docker Configuration

The application uses the official lightweight Nginx Alpine image.

## Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
```

### Docker Build

Build the Docker image:

```bash
docker build -t nginx-cicd-project:latest .
```

Check the image:

```bash
docker images
```

---

# Run Application Manually

Run the Nginx container:

```bash
docker run -d \
    --name nginx-cicd-container \
    -p 8081:80 \
    nginx-cicd-project:latest
```

Check the running container:

```bash
docker ps
```

Expected port mapping:

```text
0.0.0.0:8081->80/tcp
```

Test the application:

```bash
curl http://localhost:8081
```

Open the application in a browser:

```text
http://localhost:8081
```

Stop and remove the container:

```bash
docker rm -f nginx-cicd-container
```

---

# CI/CD Pipeline

The Jenkins pipeline automates the complete deployment process.

```text
GitHub
   |
   v
Jenkins
   |
   v
Build
   |
   v
Test
   |
   v
Docker Deploy
   |
   v
Smoke Test
   |
   v
Nginx Website
```

---

# Jenkins Pipeline Stages

## 1. Build

Jenkins builds the Docker image using the Dockerfile.

Command:

```bash
docker build -t nginx-cicd-project:latest .
```

If the Docker build fails, the pipeline stops.

---

## 2. Test

Jenkins starts a temporary Nginx container and checks whether Nginx is responding.

The test uses:

```bash
docker exec test-nginx-container \
    wget -q --spider http://127.0.0.1/
```

If Nginx responds successfully, the temporary test container is removed.

If the test fails, the deployment stage does not run.

---

## 3. Deploy

After successful testing, Jenkins starts the production container:

```bash
docker run -d \
    --name nginx-cicd-container \
    -p 8081:80 \
    nginx-cicd-project:latest
```

If an older container exists, Jenkins removes it first:

```bash
docker rm -f nginx-cicd-container || true
```

---

## 4. Smoke Test

After deployment, Jenkins verifies that the deployed website is accessible.

When Jenkins is running inside Docker on Docker Desktop, the host application is tested using:

```bash
curl -f http://host.docker.internal:8081
```

A successful response confirms that the Nginx application has been deployed correctly.

---

# Jenkinsfile

The pipeline contains the following stages:

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
```

---

# Jenkins Setup

Jenkins can be run using Docker.

Create a Docker network:

```bash
docker network create jenkins
```

Run Jenkins:

```bash
docker run -d \
    --name jenkins \
    --restart unless-stopped \
    --network jenkins \
    -p 8080:8080 \
    -p 50000:50000 \
    -v jenkins_home:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker.sock \
    jenkins/jenkins:lts
```

Check Jenkins:

```bash
docker ps
```

Jenkins is available at:

```text
http://localhost:8080
```

---

# Jenkins Job Configuration

Create a new Jenkins Pipeline job.

### Job Name

```text
nginx-cicd-pipeline-project
```

### Pipeline Definition

Select:

```text
Pipeline script from SCM
```

### SCM

Select:

```text
Git
```

### Repository

```text
https://github.com/rajesh1927/ci-cd-project-task4
```

### Branch

```text
*/main
```

### Script Path

```text
Jenkinsfile
```

Jenkins automatically checks out the repository before executing the pipeline.

---

# Pipeline Execution

When the Jenkins job is started, the following process occurs:

```text
1. Jenkins checks out code
             |
             v
2. Docker image is built
             |
             v
3. Docker image is tested
             |
             v
4. Nginx container is deployed
             |
             v
5. Deployment is smoke tested
             |
             v
6. Pipeline SUCCESS
```

---

# Expected Jenkins Result

A successful pipeline should show:

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

---

# Application Access

### Jenkins

```text
http://localhost:8080
```

### Nginx Application

```text
http://localhost:8081
```

### Test from Terminal

```bash
curl http://localhost:8081
```

---

# Troubleshooting

## Check running containers

```bash
docker ps
```

## Check Nginx container logs

```bash
docker logs nginx-cicd-container
```

## Check Nginx configuration

```bash
docker exec nginx-cicd-container nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

## Check Nginx response

```bash
curl http://localhost:8081
```

## Check Jenkins container

```bash
docker ps --filter name=jenkins
```

## Check Docker images

```bash
docker images
```

---

# CI/CD Benefits

This project demonstrates the following DevOps practices:

- Source code management using Git
- GitHub-based collaboration
- Automated CI/CD using Jenkins
- Docker containerization
- Automated application testing
- Automated deployment
- Nginx web server deployment
- Deployment verification using smoke testing

---

# Conclusion

This project successfully implements an end-to-end CI/CD pipeline for a simple static website.

The complete workflow is:

```text
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Automated Test
   ↓
Docker Deployment
   ↓
Nginx
   ↓
Smoke Test
   ↓
Website
```

The project demonstrates how Jenkins can automate the process of building, testing, and deploying a web application using Docker and Nginx.