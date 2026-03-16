# Containerizing a Java-Based Application Using CI/CD

## Overview

The developer first deploys the application locally and then deploys it to the **DEV environment**.
To automate this process, a **CI/CD pipeline** is required.

The pipeline consists of the following stages:

1. **Code**
2. **Code Quality Analysis (SonarQube)**
3. **Build (Maven)**
4. **Image** (Create Docker image)
5. **Container** (Deploy container in DEV environment)
6. **docker-compose.yml** (Run application and database containers)

The `docker-compose.yml` file creates both the **application** and **database** containers inside the DEV environment.

Once the DEV deployment is verified, the same Docker image is used to deploy the application into the **Test environment**, and testers are provided with the application URL/IP for validation.

---

## Production Pipeline

The production pipeline consists of the following stages:

1. Workspace Cleanup
2. Code
3. Code Quality Analysis (CQA)
4. Build
5. OWASP Dependency Check
6. Image
7. Trivy Scan
8. Push Image to Docker Hub
9. Deploy

> Deployment can be done using **Docker Compose**, **Docker Stack**, or **Docker Swarm**.

---

# Infrastructure & Tool Setup

## 1. Launch EC2 Instance

* Instance type: `t2.launch`
* Storage: `20 GB EBS`

---

## 2. Install Required Software

Install the following on the EC2 instance:

* Git
* Jenkins
* Docker

---

## 3. Set Up Jenkins

Complete the Jenkins initial setup and access the Jenkins dashboard.

---

## 4. Install SonarQube

Install and run SonarQube on the EC2 instance.
```
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

---

## 5. Install Trivy

Install Trivy for container image vulnerability scanning.
```
wget https://github.com/aquasecurity/trivy/releases/download/v0.18.3/trivy_0.18.3_Linux-64bit.tar.gz
tar zxvf trivy_0.18.3_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/
echo 'export PATH=$PATH:/usr/local/bin/' >> ~/.bashrc
source .bashrc
trivy --version
```

---

## 6. Create a Project in SonarQube

1. Select **Manually**
2. Enter the **Project Display Name**
3. Select **Locally**
4. Generate a **token**
5. Select **Maven**

---

## 7. Install Jenkins Plugins

Install the following plugins:

* Docker Pipeline
* Eclipse Temurin Installer
* SonarQube Scanner
* OWASP Dependency Check
* Pipeline Stage View
* Slack Notification
* NodeJS

---

## 8. Configure Slack Notifications

1. Create a Slack channel
2. Add the **Jenkins CI app** to Slack
3. Complete the integration steps

---

## 9. Configure JDK

Go to:

**Manage Jenkins → Tools**

Add JDK:

* Name: `jdk21`
* Version: `21.0.9+10`

---

## 10. Configure SonarQube Scanner

* Name: `mysonar`

---

## 11. Configure Maven

* Name: `mymaven`

---

## 12. Configure NodeJS

* Name: `node16`
* Version: `16.2.0`

---

## 13. Configure OWASP Dependency Check

* Name: `DP-Check`
* Version: `12.0.2`

---

## 14. Configure Jenkins System

Go to:

**Manage Jenkins → System**

---

## 15. Configure SonarQube Server

Add SonarQube server:

* Name: `mysonar`
* Server URL: `http://localhost:9000`
* Authentication Token: Use the token generated from SonarQube

---

## 16. Add DockerHub Credentials

Add DockerHub credentials in Jenkins.

---

## 17. Write Jenkins Pipeline

Create and configure the Jenkins pipeline script.
```bash
pipeline {
    agent any

    tools {
        maven 'mymaven'
        jdk 'jdk21'
        nodejs 'node16'
    }

    environment {
        SONARQUBE_HOME = tool('mysonar')
    }

    stages {

        stage('Clean WS') {
            steps {
                cleanWs()
            }
        }

        stage('Code') {
            steps {
                git url: 'https://github.com/kailashTuta/dockerwebapp.git'
            }
        }

        stage('CQA') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=Docker \
                        -Dsonar.host.url=http://43.205.98.214:9000 \
                        -Dsonar.login=sqa_cdc67a108c5c0aa3dbc95e742ef153b095b8f8f5
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
                sh 'cp -r target Docker-app'
            }
        }

        // stage ("OWASP FS SCAN") {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }

        stage('Docker Build') {
            steps {
                sh 'docker build -t kailashtuta/ourproject:app Docker-app'
                sh 'docker build -t kailashtuta/ourproject:db Docker-db'
            }
        }

        stage('Trivyfile') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }

        stage('ImageScan') {
            steps {
                sh 'trivy image kailashtuta/ourproject:app'
                sh 'trivy image kailashtuta/ourproject:db'
            }
        }

        stage('DockerHub') {
            steps {
                withDockerRegistry(url: 'https://index.docker.io/v1/', credentialsId: 'dockerhub') {
                    sh 'docker push kailashtuta/ourproject:app'
                    sh 'docker push kailashtuta/ourproject:db'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d'
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications'
            slackSend(
                channel: '#docker-project',
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME}\n" +
                         "Build ${env.BUILD_NUMBER}\n" +
                         "More Info: ${env.BUILD_URL}"
            )
        }
    }
}

```

---

## 18. Install Docker Compose

Run the following commands:

```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 \
-o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```

---

## 19. Run the Jenkins Pipeline

---

## 20. Access the Application

Once the pipeline completes successfully, access the deployed application using its **URL or IP address**.

---
