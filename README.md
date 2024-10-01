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


   
