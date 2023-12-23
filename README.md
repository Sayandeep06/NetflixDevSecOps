Cloud-Based Deployment of Netflix Clone using Jenkins - DevSecOps Initiative!
=============================================================================
![Alt text](<Screenshot from 2023-12-23 18-42-52.png>)
![Alt text](<Screenshot from 2023-12-23 18-47-23.png>)
### Phase 1: Initial Setup and Rollout

Step 1: Launch EC2 Instance (Ubuntu 22.04):

-   Initialize an AWS EC2 instance featuring Ubuntu 22.04.
-   Connect to the instance through SSH.

Step 2: Clone the Code:

-   Modernize all packages and subsequently clone the code.

-   Replicate your application's code repository onto the EC2 instance:

    

    `git clone https://github.com/sayandeep06/DevSecOps-Project.git`

Step 3: Docker Installation and App Deployment:

-   Configure Docker on the EC2 instance:

    

    `sudo apt-get update
    sudo apt-get install docker.io -y
    sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
    newgrp docker
    sudo chmod 777 /var/run/docker.sock`

-   Construct and run your application using Docker containers:

    
    `docker build -t netflix .
    docker run -d --name netflix -p 8081:80 netflix:latest
![Alt text](<Screenshot from 2023-12-23 18-23-44.png>)
    #to delete
    docker stop <containerid>
    docker rmi -f netflix`

An error will appear as an API key is required.
![Alt text](<Screenshot from 2023-12-23 18-49-35.png>)

Step 4: Obtain the API Key:

-   Navigate to the TMDB (The Movie Database) website using a web browser.
-   Log in, go to your profile, and access "Settings."
-   Select "API" from the left-side panel.
-   Create a new API key by choosing "Create" and accepting terms.
-   Fill in necessary details and click "Submit" to receive your TMDB API key.

Now rebuild the Docker image with your API key:



`docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .`
![Alt text](<Screenshot from 2023-12-23 18-40-46.png>)
![Alt text](<Screenshot from 2023-12-23 18-42-52-1.png>)

### Phase 2: Security

1.  SonarQube and Trivy Installation:

    -   Install SonarQube and Trivy on the EC2 instance for vulnerability scanning.

        For SonarQube:

    

        `docker run -d --name sonar -p 9000:9000 sonarqube:lts-community`

        Access it at: `publicIP:9000` (default username & password: admin)

        For Trivy:

        

        `sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy`

        Scan image using Trivy:

        

        `trivy image <imageid>`
        ![Alt text](<Screenshot from 2023-12-23 19-05-07.png>)

2.  SonarQube Integration and Configuration:

    -   Integrate SonarQube into your CI/CD pipeline.
    -   Configure SonarQube for code quality and security analysis.

### Phase 3: CI/CD Setup

1.  Jenkins Installation for Automation:

    -   Install Jenkins on the EC2 instance to automate deployment.

    Install Java:



    `sudo apt update
    sudo apt install fontconfig openjdk-17-jre
    java -version`

    Install Jenkins:

    

    `sudo wget -O /usr/share/keyrings/jenkins-keyring.asc\
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]\
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee\
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins`

    Access Jenkins at: `publicIp:8080`

2.  Essential Plugin Installation in Jenkins:

    -   Navigate to Manage Jenkins → Plugins → Available Plugins → Install plugins including Eclipse Temurin Installer, SonarQube Scanner, NodeJs Plugin, and Email Extension Plugin.

### Configuration of Java and Nodejs in Global Tool Configuration

Navigate to Manage Jenkins → Tools → Install JDK(17) and NodeJs(16) → Click on Apply and Save

### SonarQube Configuration

Create a token:

-   Go to Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text.

Configure Jenkins webhook.

### Jenkins Pipeline Configuration:

A Jenkins pipeline script is provided to automate the CI/CD process, covering steps such as cleaning workspace, code checkout, SonarQube analysis, quality gate checks, dependency installation, and Docker image creation.

For Docker login issues:


`sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins`
![Alt text](<Screenshot from 2023-12-23 21-20-41.png>)
### Phase 4: Monitoring

1.  Prometheus and Grafana Installation:

    -   Set up Prometheus and Grafana for monitoring the application.

    Prometheus Installation:

    Follow the provided steps to install Prometheus and Node Exporter on the Ubuntu instance.

    Access Prometheus at: `http://<your-prometheus-ip>:9090`

    Grafana Installation:

    Follow the provided steps to install Grafana on the Ubuntu instance.

    Access Grafana at: `http://<your-server-ip>:3000`

    Default credentials: admin/admin (Change password on first login)

    Configure Prometheus Plugin Integration:

    Modify `prometheus.yml` to include scraping metrics from Node Exporter and Jenkins.![Alt text](<Screenshot from 2023-12-23 22-30-27.png>)
    ![Alt text](<Screenshot from 2023-12-23 22-30-36.png>)
    ![Alt text](<Screenshot from 2023-12-23 22-38-50.png>)
    ![Alt text](<Screenshot from 2023-12-23 22-41-32.png>)

### Phase 5: Notification

1.  Notification Service Implementation:
    -   Set up email notifications in Jenkins or alternative notification methods.

### Phase 6: Kubernetes

1.  Kubernetes Cluster with Nodegroups:

    -   Establish a Kubernetes cluster with node groups for scalable application management.
2.  Kubernetes Monitoring with Prometheus:

 
## Monitor Kubernetes with Prometheus

Prometheus is a powerful monitoring and alerting toolkit, and you'll use it to monitor your Kubernetes cluster. Additionally, you'll install the node exporter using Helm to collect metrics from your cluster nodes.

### Install Node Exporter using Helm

To begin monitoring your Kubernetes cluster, you'll install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:

1. Add the Prometheus Community Helm repository:

    ```bash
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    ```

2. Create a Kubernetes namespace for the Node Exporter:

    ```bash
    kubectl create namespace prometheus-node-exporter
    ```

3. Install the Node Exporter using Helm:

    ```bash
    helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter
    ```

Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml:

Update your Prometheus configuration (prometheus.yml) to add a new job for scraping metrics from nodeip:9001/metrics. You can do this by adding the following configuration to your prometheus.yml file:


```
  - job_name: 'Netflix'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['node1Ip:9100']
```

Replace 'your-job-name' with a descriptive name for your job. The static_configs section specifies the targets to scrape metrics from, and in this case, it's set to nodeip:9001.

Don't forget to reload or restart Prometheus to apply these changes to your configuration.

To deploy an application with ArgoCD, you can follow these steps, which I'll outline in Markdown format:

### Deploy Application with ArgoCD

1. **Install ArgoCD:**

   You can install ArgoCD on your Kubernetes cluster by following the instructions provided in the [EKS Workshop](https://archive.eksworkshop.com/intermediate/290_argocd/install/) documentation.

2. **Set Your GitHub Repository as a Source:**

   After installing ArgoCD, you need to set up your GitHub repository as a source for your application deployment. This typically involves configuring the connection to your repository and defining the source for your ArgoCD application. The specific steps will depend on your setup and requirements.

3. **Create an ArgoCD Application:**
   - `name`: Set the name for your application.
   - `destination`: Define the destination where your application should be deployed.
   - `project`: Specify the project the application belongs to.
   - `source`: Set the source of your application, including the GitHub repository URL, revision, and the path to the application within the repository.
   - `syncPolicy`: Configure the sync policy, including automatic syncing, pruning, and self-healing.

4. **Access your Application**
   - To Access the app make sure port 30007 is open in your security group and then open a new tab paste your NodeIP:30007, your app should be running.
