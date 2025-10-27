# Spring Boot CI/CD Pipeline with Jenkins, Maven, Docker, Kubernetes, and Argo CD

This project demonstrates a complete DevOps pipeline that automates the build, test, quality analysis, packaging, deployment, and promotion of a Spring Boot application using:
   - Jenkins for Continuous Integration (CI)
   - Maven for build & dependency management
   - JUnit + Mockito for unit testing
   - SonarQube for code quality
   - Docker for containerization
   - Kubernetes + Helm for deployment
   - Argo CD for Continuous Delivery (CD)
   - AWS EC2 as the infrastructure

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

## EC2 Setup for CI/CD

#### **Note: After forking or cloning this repo make the necessary changes to the `JenkinsFile` and other files if necessary for the pipeline to function properly.**

### CI Instance:

1. Create a Ubuntu-based EC2 instance for CI.
   
2. SSH into the instance

3. Install Jenkins & Docker on the EC2 instance by using this repo: https://github.com/viz55/HelloWorld-CI-CD-with-Jenkins-Docker-GitHub.git

4. **Install SonarQube on the EC2 instance:**

   **System Requirements**
      - Java 17+ (Oracle JDK, OpenJDK, or AdoptOpenJDK)
      - Hardware Recommendations:
            - Minimum 2 GB RAM
            - 2 CPU cores
```
sudo apt update && sudo apt install unzip -y
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.4.1.88267.zip
unzip *
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-10.4.1.88267
chmod -R 775 /home/sonarqube/sonarqube-10.4.1.88267
cd /home/sonarqube/bin/linux-x86-64
./sonar.sh start
```

5. After installing jenkins, access it using this URL: `http://<ec2-instance-public-ip>:8080`

6. Configure the `docker` and `sonarqube` plugins inside jenkins UI.

7. Create a new pipeline:
    - Create a New Pipeline Job
    - Go to Jenkins dashboard → New Item → Pipeline  
    - Name: `SpringBoot-CI-CD-Pipeline`
    - Choose Pipeline type → Click OK
    - Configure GitHub Repository:
    - In Pipeline > Definition, select: `Pipeline script from SCM`
    - `SCM: Git`
    - Repository URL: `https://github.com/viz55/CI-CD-project-spring-boot-java-app.git`
    - Branch: `main`
    - Script Path: `springboot-app/JenkinsFile`

8. Access sonarqube server using this URL: `http://<EC2-Public-IP>:9000`

You will be getting an output like this after the Jenkins pipeline pushes the code to sonarqube for quality check when the CI runs:


<img width="1322" height="584" alt="Screenshot (57)" src="https://github.com/user-attachments/assets/b4178380-fe52-4b4d-86e0-eac167e7e5bd" />


9.  Create a project and generate a token for Jenkins.
10. Add this token to Jenkins credentials (ID: `sonarqube`).
11. Add Credentials in Jenkins:
     - Docker Hub → ID: `docker-cred`
     - SonarQube token → ID: `sonarqube`
     - GitHub Token → ID: `github`

The Credentials should be like this for the CI setup to work:


<img width="1027" height="284" alt="Screenshot (67)" src="https://github.com/user-attachments/assets/5bf3781c-b70b-4c1a-9c55-7c22fa9a1fa3" />


#### **Note: After running the CI setup you can check if the image was created by logging in to Docker Hub:**


<img width="1061" height="533" alt="Screenshot (58)" src="https://github.com/user-attachments/assets/91cc8c19-35ef-4bcc-b104-6a16212ea460" />


### CD Instance:

1.  Create another Ubuntu-based EC2 instance for CD.

2.  Install the following from my repo and official documentations provided:

     - Docker --> https://github.com/viz55/HelloWorld-CI-CD-with-Jenkins-Docker-GitHub.git
     - Kubectl --> https://kubernetes.io/docs/tasks/tools/install-kubectl-linux
     - Minikube --> https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download

3. Start minikube:

 ```
minikube start --driver=docker --memory=4098
 ```

4. Install Helm:

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

5. Install Argo CD via Helm:

```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm upgrade --install argocd argo/argo-cd -n argocd
```

To check whether argo-cd service has been deployed or not run:

```
kubectl get svc -n argocd
```

You should be getting the output like this:


<img width="915" height="111" alt="Screenshot (66)-get svc" src="https://github.com/user-attachments/assets/6a27a26b-4e52-47dc-a33f-add33b4fb082" />


6. Access Argo CD UI:

```
kubectl port-forward svc/my-argocd-server -n argocd --address 0.0.0.0 8080:443
```

<img width="850" height="202" alt="Screenshot (66)-argo deploy" src="https://github.com/user-attachments/assets/de139ac1-ab01-4970-9d7e-fa2541c918d5" />


Then open in browser to access the Argo-CD UI: `https://<EC2-Public-IP>:8080`


7. Get Argo CD admin Password to login by using this command:

```
kubectl -n argocd get secret my-argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

8. **Create Application in Argo CD:**

 In the Argo CD UI provide the following details:
   - Application Name: `springboot-app`
   - Repository URL: `https://github.com/viz55/CI-CD-project-spring-boot-java-app`
   - Path: `spring-boot-app-manifests`
   - Cluster URL: `https://kubernetes.default.svc`
   - Namespace: `default`

You should be entering the details like this:-


<img width="985" height="566" alt="Screenshot (63)" src="https://github.com/user-attachments/assets/6f89aa4f-cf2f-4770-b9eb-1458e7e58069" />


After the manifest file gets synced the output should look like this:


<img width="1345" height="625" alt="Screenshot (62)" src="https://github.com/user-attachments/assets/9ebdf89d-a571-4294-922f-127f166877be" />


The deployment manifest gets synced and the deployed app can be accessed by using this command in a new terminal window:

```
kubectl port-forward svc/springboot-app-service -n default --address 0.0.0.0 9100:80 
```


<img width="899" height="185" alt="Screenshot (65)" src="https://github.com/user-attachments/assets/833f4e20-3301-478b-afc3-1f56148b6e5f" />


The port forwarding will occur and the deployed website can be accessed using: `http://<ec2-instance-ip>:9100`


The output of the deployed application will be:


<img width="1299" height="445" alt="Screenshot (64)" src="https://github.com/user-attachments/assets/8beb1182-ab76-4e10-a569-f7ee016cfd5f" />


## Pipeline workflow:-

1. Trigger the Jenkins pipeline manually or via webhook to start the CI/CD process for the Java application.
2. Jenkins uses:
      - the `Git plugin` to check out the source code from the Git repository.
      - the `Maven Integration plugin` to package the application into a `JAR file`, Downloads dependencies from `pom.xml` & prepares the build.
      - the `JUnit` and `Mockito` plugins to run unit tests.
      - the `SonarQube` plugin to analyze the code quality of the Java application.
      - the `Docker plugin` uses the Dockerfile inside the repository to build an image of the Spring Boot app. The app’s `JAR file` (from Maven build) is packaged into the image & Pushes the new image to Docker Hub using the provided credentials.

3. Jenkins updates Deployment Manifest (deployment.yaml) with the new image tag. It also Commits and pushes this change to the GitHub repo. This change triggers Argo CD to detect and sync the update.
4. Helm applies updated Kubernetes manifests & the Spring Boot app is deployed to `Minikube` test environment.
5. Jenkins triggers Argo CD CLI to sync the latest manifest & Argo CD automatically pulls from Git and updates production cluster.

## Credits & Acknowledgements:-

This project was inspired and guided by the teachings of [Abhishek Veeramalla](https://github.com/iam-veeramalla) whose tutorials and DevOps content helped me understand and implement an end-to-end CI/CD pipeline using Jenkins, Docker, SonarQube, Kubernetes, Helm, and Argo CD.

Special thanks to:

- The **Open Source Community** for the amazing DevOps tools.
- **You**, for checking out this project! 💙

**Author:** Vishnu M V  
**GitHub:** [@viz55](https://github.com/viz55)

**If you find this project helpful, please ⭐ star the repository.**

## 📄 License

This project is licensed under the [MIT License](LICENSE) — you’re free to use, modify, and distribute this code with attribution.
