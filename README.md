# End-to-End DevOps Pipeline: Building, Testing, and Deploying with Jenkins

## Project Overveiw

![Untitled-fotor-20240929171830](https://github.com/user-attachments/assets/278cae07-c4d0-492a-850e-62f70bbddc1d)


This project demonstrates a complete end-to-end CI/CD pipeline for deploying a microservice-based application in Kubernetes, using industry-standard DevOps tools and practices. The pipeline is fully automated and integrates with GitHub for source code version control, Jenkins for continuous integration and deployment, and Kubernetes for container orchestration.

Key stages include Maven for build and test automation, SonarQube for code quality analysis, and Trivy for vulnerability scanning of Docker images. The Dockerized application is stored in Nexus repository before being securely deployed to Kubernetes, with additional security scanning by Kubeaudit. Continuous monitoring and alerting are handled by Prometheus and Grafana, ensuring robust application performance and security insights.

This project showcases modern DevOps practices like infrastructure-as-code (IaC), security integration (DevSecOps), and continuous monitoring, making it highly scalable and production-ready. It reflects my expertise in streamlining software delivery processes while ensuring code quality, security, and operational efficiency.

## Main Features

1. Continuous Integration and Continuous Deployment (CI/CD) with Jenkins
   - Automated pipeline built with Jenkins to manage the complete software development lifecycle, 
     from code commit to deployment.
  
2. Maven Build and Testing
   - Automated build and test process using Maven to ensure code quality and functionality before 
     deployment.
    
3. Code Quality Analysis with SonarQube
   - Static code analysis performed by SonarQube to detect bugs, vulnerabilities, and code smells, 
     ensuring high-quality code.
  
4. Container Image Scanning with Trivy
   - Trivy scans Docker images for vulnerabilities to enhance security before pushing them to the 
      registry.

5. Artifact Management with Nexus Repository
   - Maven-generated artifacts are stored and managed in Nexus Repository, ensuring a centralized, 
     version-controlled storage.

6. Dockerized Application Deployment to Kubernetes
   - Fully Dockerized application is built, tagged, and pushed to Docker Hub, followed by deployment 
     to a Kubernetes cluster for container orchestration.
  
7. Kubernetes Security Scanning with Kubeaudit
   - Kubeaudit scans the Kubernetes deployment for potential security issues and provides         
     recommendations to enhance the security of the environment.

8. Prometheus and Grafana for Monitoring
   - Prometheus monitors system performance and resource usage, while Grafana provides real-time 
     visualization and alerts for the application’s health.

9. Email Notifications for Pipeline Events
   -  Configured Jenkins to send email alerts for successful or failed pipeline stages, ensuring 
      real-time status updates.

10. Scalable and Secure Infrastructure
    - The project incorporates scalable Kubernetes architecture and security best practices, 
      ensuring a production-ready deployment environment.

## Technologies Used

* **Jenkins**: For automating the CI/CD pipeline, handling tasks such as building, testing, and deploying the application.

* **Maven**: Used to automate the build process, manage dependencies, and execute unit tests.

* **SonarQube**: For performing static code analysis to ensure the code is free from bugs, vulnerabilities, and code smells.

* **Trivy**: A security scanner that checks Docker images for vulnerabilities before pushing them to a registry.

* **Nexus Repository**: A repository manager used to store and version control Maven-generated artifacts and Docker images.

* **Docker**: For containerizing the application, enabling it to run consistently across different environments.

* **Kubernetes**: Used to deploy, manage, and scale the Dockerized application in a cluster environment.

* **Prometheus**: Provides real-time monitoring and alerting for application and infrastructure metrics.

* **Grafana**: Used for creating dashboards to visualize metrics from Prometheus and send alerts based on performance or health thresholds.

* **GitHub**: Acts as the source code repository and triggers Jenkins when code is committed or pushed.


## Project Demonstration

### 1. Launching EC2 instances
   - To set up a Kubernetes cluster, you need to launch three EC2 instances: one master and two slave nodes:
     
   - Choose **t2.medium** as the instance type for all three nodes. This instance type has enough resources for a small cluster.
     
   - Network: Select the project-2 network setting which i have already configured which is shown in the below image

   - Review your configurations and click on the Launch button.
  

![Screenshot (299)](https://github.com/user-attachments/assets/cff7bf9b-e68c-4789-b160-20bdc0bd22b0)


![Screenshot (292)](https://github.com/user-attachments/assets/1bcba331-f1b1-4522-a55f-2ef97535171f)


### 2. Installing Docker and Kubernetes on the servers
   - SSH into the Instances or you can directly connect using EC2 instance connect.
   - Firsly you have to shift to the root user using sudo su.
  
   ### Install Kubernetes(kubeadm, kubectl, kubelet) version 1.28.1:
   - Update the package list and install docker:
   ```
   sudo apt-get update
   sudo apt-get install -y docker.io
   
   ```

   ![Screenshot (302)](https://github.com/user-attachments/assets/798b7a3d-bfce-429c-ab77-5c16f2b52db6)


   ![Screenshot (303)](https://github.com/user-attachments/assets/93c27e5b-2e23-4a88-aba6-21af4297b26c)


   - Install apt transport, certificates, and curl:
   ```
   sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
   ```

   - Add the Kubernetes apt repository:
   ```
   curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   ```

   - Install kubeadm, kubelet, and kubectl
   ```
   sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-2.1 kubectl=1.28.1-1.1
   ```

   - On the master node, run the following command to initialize the Kubernetes cluster:
   ```
   sudo kubeadm init --pod-network-cidr=10.244.0.0/16
   ```

   - After successful initialization, kubeadm will output a join command. It will look something 
   like this:

   ```
   kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```
   Copy this command, as you will need it to add the worker nodes (slaves) to the cluster later.

   
   ![Screenshot (309)](https://github.com/user-attachments/assets/5c92e007-adc3-4006-88cd-0ee02dcd909a)


   ![Screenshot (310)](https://github.com/user-attachments/assets/e81bda80-71d1-496d-81d8-7d03c9feda6f)


   - To use kubectl on the master node, you need to set up the kubeconfig file for the current user:
   
   ```
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

   ![Screenshot (313)](https://github.com/user-attachments/assets/c18f39c2-81b5-4f2a-b8b4-a40d4a6a42d8)


   ### 3. Installing Calico Network plugin

   - Use kubectl to apply the Calico manifest. The following command pulls the latest Calico installation manifest from the official Project Calico documentation:
   
   ```
   kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
   ```

   - Once Calico is installed and the pods are running, you can verify the node status. The master node should now be in the "Ready" state:
  
   ```
   kubectl get nodes
   ```

   ![Screenshot (314)](https://github.com/user-attachments/assets/7f4a6bc5-6828-4e7b-8b50-3acfd7ec8f18)

   
   ![Screenshot (316)](https://github.com/user-attachments/assets/d269be7a-b00f-4c20-8772-81597b6c2598)

   ### 4. Launching Two Instances for SonarQube and Nexus

   - Launch two t2.medium EC2 instances (or other instance types as per your requirements) for SonarQube and Nexus.

     
   ![Screenshot (318)](https://github.com/user-attachments/assets/8ae14bb6-a257-4281-8496-b134eee418be)


   - After logging into each instance, update the package list to ensure the latest packages are installed:

   ```
   sudo apt-get update
   ```

   - Install required packages that allow apt to work with repositories over HTTPS:

   ```
   sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
   ```

   - Add Docker’s official GPG key to the system:

   ```
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o          
   /usr/share/keyrings/docker-archive-keyring.gpg
   ```

   - Add the Docker repository for Ubuntu:

   ```
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

   - Install Docker and its dependencies:

   ```
   sudo apt-get install -y docker-ce docker-ce-cli containerd.io
   ```

   
   ![Screenshot (319)](https://github.com/user-attachments/assets/785744ab-c919-4370-94d4-7a094231cb3f)

   
   ![Screenshot (320)](https://github.com/user-attachments/assets/46300c5c-1250-4dab-9b8e-2afd9f8ccf21)


   ![Screenshot (321)](https://github.com/user-attachments/assets/194ff4ff-56df-406d-b71d-e828bde99993)


   ![Screenshot (322)](https://github.com/user-attachments/assets/abd11af4-4476-4b69-8e4f-c092c5a2907b)



   ### 5. Running SonarQube and Nexus Docker Images on Respective Servers

   - After Docker has been installed on the SonarQube instance, you can pull and run the SonarQube Docker image as follows:

   ```
   docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
   ```


   - For the Nexus server, follow similar steps to pull and run the Nexus Docker image:

   ```
   docker run -d --name nexus -p 8081:8081 -p 8082:8082 sonatype/nexus3
   ```


   ![Screenshot (323)](https://github.com/user-attachments/assets/03ba95d1-3205-431b-b80e-0235542c7de2)


   ![Screenshot (324)](https://github.com/user-attachments/assets/efb8c70c-2cc3-43d5-b312-7d208ae419fd)


   ### 6. Logging into SonarQube and Nexus

   - Logging into SonarQube: Open a web browser and navigate to http://<sonarqube-server-public-ip>:9000
   - Default Credentials:
         Username: admin
         Password: admin

   - Logging into Nexus Repository Manager: Open a web browser and navigate to http://<nexus-server-public-ip>:8081.

   - Default Credentials:
           Username: admin
           Password: You will need to get the initial admin password from inside the Nexus          
 container. Run the following command to retrieve it:

   ```
   sudo docker exec -it nexus cat /nexus-data/admin.password
   ```

  ![Screenshot (327)](https://github.com/user-attachments/assets/1c0ee78d-3e50-4712-b1ef-51d400af2d6e)

   
   ![Screenshot (330)](https://github.com/user-attachments/assets/ff547b57-89ae-43dc-bcdb-30873fc63e5f)


   ![Screenshot (328)](https://github.com/user-attachments/assets/ef36164a-b495-4e8e-8662-6aa1e64454f9)


   ### 7. Launching the instance for Jenkins 

   - Launch a **t2.medium** instance (or as per your need) with 20 GB storage for the Jenkins server.
   - Step-by-Step Jenkins Installation:
     - Update the package list
     - Install Java(required for java)
     - Add Jenkins Repository Key
     - Add Jenkins Package Repository
     - Install Jenkins
     - sudo systemctl enable jenkins
     - sudo systemctl start jenkins

   ```
   sudo apt-get update

   sudo apt-get install -y openjdk-11-jdk

   curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
   /usr/share/keyrings/jenkins-archive-keyring.asc > /dev/null

   echo deb [signed-by=/usr/share/keyrings/jenkins-archive-keyring.asc] \
   https://pkg.jenkins.io/debian binary/ | sudo tee \
   /etc/apt/sources.list.d/jenkins.list > /dev/null

   sudo apt-get install -y jenkins

   sudo systemctl enable jenkins
   sudo systemctl start jenkins
   ```
   - To unlock Jenkins, retrieve the initial admin password from the server:

   ```
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

   ![Screenshot (331)](https://github.com/user-attachments/assets/91e98317-a72c-43e0-b718-6d4909334b07)

       
   ![Screenshot (332)](https://github.com/user-attachments/assets/807e3f38-001b-44f1-86a3-ba05c9173684)

   
  ![Screenshot (334)](https://github.com/user-attachments/assets/4a351926-01b5-4eca-8f8f-77c636e454da)

   
   ### 8. Installing Essential Jenkins Plugins

   - Search and install the following plugins:
     - Eclipse Temurin
     - Maven Integration
     - SonarQube Scanner
     - Docker Pipeline
     - Kubernetes
     - Config file provider
     - Docker
   
   - After selecting all the required plugins, click on the Install without Restart button. Jenkins will begin downloading and installing the plugins.


   ![Screenshot (335)](https://github.com/user-attachments/assets/5ace3849-3ea4-4935-b269-17734d2aa650)


   ### 9. Configuring Tools in Jenkins

   -   After installing the necessary plugins, the next step is to configure the tools like JDK, Maven, SonarQube, Docker, and Git in Jenkins.

   1.  Configuring JDK:
       - Manage Jenkins → Global Tool Configuration.
       - Scroll down to the JDK section.
       - Click Add JDK.
       - Install automatically: Check this option.

   2. Configuring Maven:
         - In the Global Tool Configuration page, scroll to Maven.
         - Click Add Maven.
         - Install automatically: Check this option.

   3. Configuring SonarQube:
         - Scroll to SonarQube Servers.
         - Click **Add SonarQube**.
         - Server URL: Provide your SonarQube server URL (e.g., http://<sonarqube-server-ip>:9000).
         - **Server Authentication Token**: Add the authentication token generated from your SonarQube server.

   ![Screenshot (344)](https://github.com/user-attachments/assets/62915d02-3cc8-4fb3-8b23-0c804308f226)

   ![Screenshot (338)](https://github.com/user-attachments/assets/4a7fb032-354d-4a1e-b096-1c645cf9c235)


   ### 10. Starting a New Project with Jenkins Pipeline

   - From your Jenkins home page, click on New Item.
   - Give your project a meaningful name (e.g., Java-Game-Deploy).
   - Select **Pipeline** from the list of project types and click OK.

   ![Screenshot (341)](https://github.com/user-attachments/assets/1516b266-0457-49fd-8823-926e9c4e6bcd)


   ### 11. Adding Nexus Repositories to distributionManagement in pom.xml

   - Adding Nexus Repositories to distributionManagement in pom.xml
   
   1. Open the pom.xml File.
   2. Add the following <distributionManagement> section to specify where Maven will deploy the release and snapshot artifacts.

   ```
   <distributionManagement>
        <!-- Releases Repository Configuration -->
        <repository>
            <id>nexus-releases</id>
            <url>http://<nexus-server-ip>:8081/repository/maven-releases/</url>
        </repository>

        <!-- Snapshots Repository Configuration -->
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <url>http://<nexus-server-ip>:8081/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>
   ```
   3. Save and Commit Your Changes


   ![Screenshot (346)](https://github.com/user-attachments/assets/4fabda50-b024-45f7-8153-f5a6f06f5aba)


   ![Screenshot (347)](https://github.com/user-attachments/assets/a1be454f-4c82-48b0-8547-2f2d6fa93ddf)


   
   ### 12. Configuring Global Maven settings.xml with Server ID, Username, and Password

   To securely manage your credentials (like Nexus username and password) for deploying artifacts, you need to configure the global Maven settings.xml file. This file contains the authentication details (such as the server id, username, and password) required for deployment.

  Here’s how you can set it up:
  
  1. Locate the Global settings.xml File.
   - If the **settings.xml** file does not exist in this location, you can create a new one.

   ![Screenshot (350)](https://github.com/user-attachments/assets/c6ee5309-44ae-4f7b-945f-3267d6f69907)


   2. Add your Nexus server details, including the server id, username, and password, inside the       <servers> tag in the settings.xml file. Here is an example:

   ```
   <!-- Servers Section -->
    <servers>
        <!-- Nexus Releases Repository Credentials -->
        <server>
            <id>nexus-releases</id>
            <username>your-username</username>
            <password>your-password</password>
        </server>

        <!-- Nexus Snapshots Repository Credentials -->
        <server>
            <id>nexus-snapshots</id>
            <username>your-username</username>
            <password>your-password</password>
        </server>
    </servers>
   ```

   ![Screenshot (352)](https://github.com/user-attachments/assets/5299179d-5aec-4fd9-bd7d-55d72e7a616b)


   ## Writing a Jenkins Pipeline for CI/CD:

   To implement your CI/CD pipeline in Jenkins, you need to define a Jenkins Pipeline that automates each of the steps in your project flow, such as code checkout, build, test, analysis, packaging, and deployment.


- Here's my Jenkins Declarative Pipeline which i have used in the tutorial:

```

pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    enviornment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/jaiswaladi246/Boardgame.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analsyis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
        
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }
        
        stage('Publish To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t adijaiswal/boardshack:latest ."
                    }
               }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html adijaiswal/boardshack:latest "
            }
        }
        
        stage('Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker push adijaiswal/boardshack:latest"
                    }
               }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
                        sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
        
        stage('Verify the Deployment') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
                        sh "kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
                }
            }
        }
        
        
    }
    post {
    always {
        script {
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            def body = """
                <html>
                <body>
                <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                <h2>${jobName} - Build ${buildNumber}</h2>
                <div style="background-color: ${bannerColor}; padding: 10px;">
                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                </div>
                <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                </div>
                </body>
                </html>
            """

            emailext (
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'katiyarsatyak4@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
}

}
```
  
- Here's a brief explanation of the steps in our Jenkins pipeline:

1. Git Checkout:

   - Pulls the code from the main branch of your GitHub repository using the provided credentials (git-cred).

2. Compile:

   - Runs the mvn compile command to compile the code using Maven.
  
3. Test:

   - Executes the mvn test command to run the unit tests.

4. File System Scan:

   - Uses Trivy to scan the file system for vulnerabilities and outputs the report in HTML format (trivy-fs-report.html).

5. SonarQube Analysis:

   - Performs static code analysis using SonarQube with the provided project name and key.

6. Quality Gate:

   - Waits for SonarQube's Quality Gate result. It ensures that the project meets the required quality standards.

7. Build:

   - Runs the mvn package command to package the application into a deployable format (e.g., JAR or WAR file).

8. Publish to Nexus:

   - Deploys the built artifact (packaged app) to a Nexus repository using Maven.
9. Build & Tag Docker Image:

   - Builds a Docker image of the application and tags it as latest.

10. Docker Image Scan:

    - Uses Trivy to scan the Docker image for vulnerabilities and outputs the report in HTML format (trivy-image-report.html).

11. Push Docker Image:

   - Pushes the Docker image to a Docker registry.

12. Deploy to Kubernetes:

   - Deploys the application to a Kubernetes cluster using the kubectl apply command with the specified deployment YAML file.
13. Verify the Deployment:

   - Verifies the deployment by checking the status of the pods and services in the Kubernetes webapps namespace.

14. Post-Stage (Notification):

   - Sends an email notification with the pipeline status (success/failure) and attaches the Trivy scan report. It uses Jenkins environment variables to customize the message with job details.


   
   -- Each of these stages helps ensure that the code is built, tested, scanned for vulnerabilities, deployed to a Kubernetes cluster, and monitored, making it a comprehensive CI/CD pipeline for deploying a secure, production-ready application.

    
![Screenshot (354)](https://github.com/user-attachments/assets/6d8673a9-ef02-4b1e-bfbb-f1e83203f01f)



![Screenshot (355)](https://github.com/user-attachments/assets/edd9cf8d-ce9f-4725-968e-14c5eb9e4dd6)


   
### Creating service YAML file:

1. Creating svc.yml file:
   - This file defines the service configuration in Kubernetes to expose your application to the network.
  
2. Run kubectl create ns webapps:
   - This command creates a new namespace called webapps in your Kubernetes cluster. Namespaces are used to organize objects in Kubernetes, providing a way to group services, pods, and other resources.
  
3. Run kubectl apply -f svc.yml:
   - This command applies the svc.yml configuration to the cluster. It creates the service defined in the YAML file in the webapps namespace. This service exposes your application to the network as defined by the service type i.e LoadBalancer.
  

-- These steps are setting up the networking and exposure of your application in Kubernetes


   ![Screenshot (358)](https://github.com/user-attachments/assets/a94e86e2-71d6-412d-9163-17f42356220d)


   ### Creating the role.yml file:

   - You’ll use a text editor (e.g., vi) to create a YAML file that defines the roles and permissions for a user or group of users in Kubernetes.
   - Roles define what actions are allowed within a specific namespace.

![Screenshot (359)](https://github.com/user-attachments/assets/6483b8a5-33f0-45f4-88b3-eb60bf2c3f3e)


![Screenshot (362)](https://github.com/user-attachments/assets/2a07bf7f-0b88-4110-a71e-0b4c4d8cbc19)


- 2. kubectl apply -f role.yml:
   - This command applies the role configuration in the role.yml file to the webapps namespace. It creates a role named webapps-role, giving specific permissions (like get, create, update, etc.) to resources like pods, services, and deployments within the namespace.
 
- 3. Binding the Role:
     - You also need to bind the role to a user or service account, you would create a RoleBinding file (e.g., role-binding.yml) and apply it as well:


![Screenshot (363)](https://github.com/user-attachments/assets/7980c1ec-af5d-4abb-98f3-a35ab31906f1)

   ![Screenshot (366)](https://github.com/user-attachments/assets/988949f9-5162-4cc1-9e76-698cf9749aba)


![Screenshot (367)](https://github.com/user-attachments/assets/9dd1203f-8b80-40ef-9e1c-8ceb65f5b738)



## Creating the deployment-service.yml in GitHub Repository

-- This step involves defining the deployment and service configuration for your Kubernetes cluster, and then storing it in your GitHub repository.

1. Create the deployment-service.yml File:

```
apiVersion: apps/v1
kind: Deployment # Kubernetes resource kind we are creating
metadata:
  name: java-game-deployment
spec:
  selector:
    matchLabels:
      app: javagamedeploy
  replicas: 2 # Number of replicas that will be created for this deployment
  template:
    metadata:
      labels:
        app: javagamedeploy
    spec:
      containers:
        - name: javagamedeploy
          image: samy2908/javagamedeploy:latest # Image that will be used to containers in the cluster
          imagePullPolicy: Always
          ports:
            - containerPort: 8080 # The port that the container is running on in the cluster


---

apiVersion: v1 # Kubernetes API version
kind: Service # Kubernetes resource kind we are creating
metadata: # Metadata of the resource kind we are creating
  name: Java-Deploy-k8
spec:
  selector:
    app: javagamedeploy
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 8080 
  type: LoadBalancer 
```


![Screenshot (373)](https://github.com/user-attachments/assets/b9367833-db64-403a-98fc-0a1e6e100259)


## Setting Up Extended Email Notifications in Jenkins.

1. Generate Google App Password:
   - To allow Jenkins to send email notifications using your Gmail account, you need to generate an App Password for Jenkins to use.

2. Configure Extended Email Notification Plugin in Jenkins:
   - Search for and install the Email Extension Plugin (also known as Extended Email Notification Plugin).
  
3. Set Up SMTP Server in Jenkins:
   - Go to Manage Jenkins > Configure System.
   - Scroll down to the Extended E-mail Notification section.
   - Fill in the SMTP server settings:
      - SMTP Server: smtp.gmail.com
      - Default user e-mail suffix: Your-email@gmail.com
      - Use SMTP Authentication: Checked
      - SMTP Username: Your Gmail email (e.g., yourname@gmail.com)
      - SMTP Password: Use the Google App Password you generated earlier.
      - SMTP Port: 465
      - Use SSL: Checked
        
4. Test Email Notification Configuration:
   - In the Extended E-mail Notification section, there's an option to test the email configuration.
   -  Fill in a test recipient email and click "Test configuration by sending test e-mail." If the email is received, the setup is correct.

5. Configure Email Notifications in Your Pipeline:
   - In your pipeline, you need to configure email notifications to be triggered based on the build status.


![Screenshot (378)](https://github.com/user-attachments/assets/0106dcce-59eb-41e4-b4e5-e7923fdd4efb)

   
### Connecting SonarQube to Jenkins Using Webhooks

To integrate SonarQube with Jenkins using webhooks, the goal is to allow SonarQube to notify Jenkins when the analysis is completed. Here's how you can set up this integration:

1. Login to your SonarQube instance as an admin.
2. Go to Administration > Project Settings > Webhooks.
3. Add a new webhook:
   - Name: Enter a name (e.g., Jenkins Webhook).
   - Click **Create**.
  

![Screenshot (380)](https://github.com/user-attachments/assets/05641b52-e6be-4a3a-8756-70968f9ec96f)


### Building the Project:

   - After completing all the steps, you are now ready to build and deploy the project. The build and deployment process involves compiling the code, packaging it, and then deploying the application to the Kubernetes cluster.

   - The game was successfully deployed and accessible at port 32399 on my Kubernetes cluster. To access the game:
Use the external IP or node IP of your Kubernetes cluster along with port 32399.
Open a browser and enter **http://<Node_IP>:32399** to play the game.


![Screenshot (382)](https://github.com/user-attachments/assets/933594b1-e33b-490b-aeee-02252ad83fc4)


![Screenshot (383)](https://github.com/user-attachments/assets/9ba113b7-e89c-4ca5-8ec8-a93863052319)



## Deploying Prometheus and Grafana on your monitoring server:

- Deploy a new instance with 15 GB of storage.
- Pull the Prometheus Docker image.
- Run the Prometheus container.
```
sudo apt-get update
sudo apt-get install -y docker.io
sudo docker pull prom/prometheus
sudo docker run -d --name prometheus -p 9090:9090 prom/prometheus
```


![Screenshot (384)](https://github.com/user-attachments/assets/bac224f0-7759-42da-8395-495552d8fd02)


![Screenshot (390)](https://github.com/user-attachments/assets/cc683528-e44b-4583-88b5-e294791cea07)


## Install Grafana:

1. Let's install granfana in another which i have used in my tutorial:
   - Update your system packages.
   - sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
   - echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
   - sudo apt-get install -y grafana


![Screenshot (391)](https://github.com/user-attachments/assets/a17adac3-d027-42e8-8776-50c90c2595ce)

![Screenshot (393)](https://github.com/user-attachments/assets/8332ea6e-0c26-49d4-8517-f6b4f0402b73)


![Screenshot (394)](https://github.com/user-attachments/assets/c9de1755-3c85-4424-8872-74eb34f2de51)

![Screenshot (395)](https://github.com/user-attachments/assets/0755cca0-239f-42ae-be01-3973ae111f53)

![Screenshot (396)](https://github.com/user-attachments/assets/2555dba4-1673-47af-824d-73553ad38c21)









   







   

