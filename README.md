Cloud-Based Deployment of Netflix Clone using Jenkins - DevSecOps Initiative!
=============================================================================

### Phase 1: Initial Setup and Rollout

Step 1: Launch EC2 Instance (Ubuntu 22.04):

-   Initialize an AWS EC2 instance featuring Ubuntu 22.04.
-   Connect to the instance through SSH.

Step 2: Clone the Code:

-   Modernize all packages and subsequently clone the code.

-   Replicate your application's code repository onto the EC2 instance:

    bashCopy code

    `git clone https://github.com/sayandeep06/DevSecOps-Project.git`

Step 3: Docker Installation and App Deployment:

-   Configure Docker on the EC2 instance:

    bashCopy code

    `sudo apt-get update
    sudo apt-get install docker.io -y
    sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
    newgrp docker
    sudo chmod 777 /var/run/docker.sock`

-   Construct and run your application using Docker containers:

    bashCopy code

    `docker build -t netflix .
    docker run -d --name netflix -p 8081:80 netflix:latest

    #to delete
    docker stop <containerid>
    docker rmi -f netflix`

An error will appear as an API key is required.

Step 4: Obtain the API Key:

-   Navigate to the TMDB (The Movie Database) website using a web browser.
-   Log in, go to your profile, and access "Settings."
-   Select "API" from the left-side panel.
-   Create a new API key by choosing "Create" and accepting terms.
-   Fill in necessary details and click "Submit" to receive your TMDB API key.

Now rebuild the Docker image with your API key:

bashCopy code

`docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .`

### Phase 2: Security

1.  SonarQube and Trivy Installation:

    -   Install SonarQube and Trivy on the EC2 instance for vulnerability scanning.

        For SonarQube:

        bashCopy code

        `docker run -d --name sonar -p 9000:9000 sonarqube:lts-community`

        Access it at: `publicIP:9000` (default username & password: admin)

        For Trivy:

        bashCopy code

        `sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy`

        Scan image using Trivy:

        bashCopy code

        `trivy image <imageid>`

2.  SonarQube Integration and Configuration:

    -   Integrate SonarQube into your CI/CD pipeline.
    -   Configure SonarQube for code quality and security analysis.

### Phase 3: CI/CD Setup

1.  Jenkins Installation for Automation:

    -   Install Jenkins on the EC2 instance to automate deployment.

    Install Java:

    bashCopy code

    `sudo apt update
    sudo apt install fontconfig openjdk-17-jre
    java -version`

    Install Jenkins:

    bashCopy code

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

bashCopy code

`sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins`

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

    Modify `prometheus.yml` to include scraping metrics from Node Exporter and Jenkins.

### Phase 5: Notification

1.  Notification Service Implementation:
    -   Set up email notifications in Jenkins or alternative notification methods.

### Phase 6: Kubernetes

1.  Kubernetes Cluster with Nodegroups:

    -   Establish a Kubernetes cluster with node groups for scalable application management.
2.  Kubernetes Monitoring with Prometheus:

    -   Integrate Prometheus to monitor the Kubernetes cluster.
    -   Install Node Exporter via Helm to collect metrics from cluster nodes.

    Node Exporter Helm Installation:

    bashCopy code

    `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    kubectl create namespace prometheus-node-exporter
    helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter`

    Add a job to scrape metrics on nodeip:9001/metrics in prometheus.yml.

### Phase 7: Cleanup

1.  AWS EC2 Instance Cleanup:
    -   Terminate unnecessary AWS EC2 instances.

These steps outline the deployment of a Netflix Clone on the Cloud using Jenkins as part of a comprehensive DevSecOps project.