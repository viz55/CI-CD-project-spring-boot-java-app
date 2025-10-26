# Spring Boot CI/CD Pipeline with Jenkins, Maven, Docker, Kubernetes, and Argo CD

## Project Overview

- **CI Instance**: Runs Jenkins, Maven, Docker, and SonarQube to build, test, analyze, and package the application.  
- **CD Instance**: Runs Kubernetes (Minikube), Argo CD, and Helm to deploy and manage applications in a GitOps workflow.

The pipeline automates the **entire lifecycle**: code checkout → build → test → code analysis → Docker image build → deployment to Kubernetes → GitOps-based promotion to production.

## Technologies Used

- **Spring Boot** – Java framework  
- **Maven** – Build, test, package, and deploy Spring Boot application  
- **JUnit & Mockito** – Unit testing  
- **SonarQube** – Code quality analysis  
- **Docker** – Containerization  
- **Kubernetes (Minikube)** – Cluster orchestration  
- **Helm** – Kubernetes package manager  
- **Argo CD** – GitOps deployment tool  
- **Jenkins** – CI/CD automation  
- **AWS EC2** – Hosting CI and CD environments

## EC2 Setup

### CI Instance

1. Ubuntu-based EC2 instance.  
2. Install Jenkins & Docker on the EC2 instance by using this repo: https://github.com/viz55/HelloWorld-CI-CD-with-Jenkins-Docker-GitHub.git 
3. **Install SonarQube on the EC2 instance:**

```
System Requirements
Java 17+ (Oracle JDK, OpenJDK, or AdoptOpenJDK)
Hardware Recommendations:
   Minimum 2 GB RAM
   2 CPU cores
sudo apt update && sudo apt install unzip -y
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip *
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-10.4.1.88267
chmod -R 775 /home/sonarqube/sonarqube-10.4.1.88267
cd /home/sonarqube/bin/linux-x86-64
./sonar.sh start
```

4. After installing jenkins, access it using the `http://<ec2-instance-public-ip>:8080` 
5.  
