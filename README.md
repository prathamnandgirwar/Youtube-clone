![WhatsApp Image 2024-11-26 at 01 37 57](https://github.com/user-attachments/assets/55fec01a-b972-4f12-a659-7d48f1aaf1d5)



# $${\color{Red} \textbf {Deploy Youtube Clone on Cloud using Jenkins - DevOps Project!}}$$


## $${\color {red} \textbf {Phase 1: Initial Setup and Deployment}}$$

**Step 1: Launch EC2 (Ubuntu 22.04):**

- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.

**Step 2: Clone the Code:**

- Update all the packages and then clone the code.
- Clone your application's code repository onto the EC2 instance:
    
    ```bash
    git clone https://github.com/prathamnandgirwar/Youtube-clone.git
    ```
    

**Step 3: Install Docker and Run the App Using a Container:**

- Set up Docker on the EC2 instance:
    
    ```bash
    
    sudo apt-get update
    sudo apt-get install docker.io -y
    sudo systemctl start docker
    sudo usermod -aG docker ubuntu
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
    ```

 **Step 3: Install Terraform :**

- Set up Terraform on the EC2 instance:
  
 ````
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt-get install terraform
terraform --version

````

${\color{blue} \textbf {Setup  AWS CLI:}}$
````
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip 
unzip awscliv2.zip
sudo ./aws/install
aws --version

````

 **Step 5: Install Kubernetes:**
 ## ${\color{blue} \textbf {Install kubectl}}$
Download the latest release with the command:
````
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
````
Validate the binary 
````
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
````
Validate the kubectl binary against the checksum file:
````
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
````
Install kubectl:
````
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
````
Note:
If you do not have root access on the target system, you can still install kubectl to the ~/.local/bin directory:
````
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
````
````
kubectl version --client
````

## $${\color {red} \textbf {Phase 2: Security}}$$

1. **Install SonarQube and Trivy:**
    - Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.
        
        sonarqube
        ```
        docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
        ```
        
        
        To access: 
        
        publicIP:9000 (by default username & password is admin)
        
        To install Trivy:
        ```
        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy        
        ```
        
        
2. **Integrate SonarQube and Configure:**
    - Integrate SonarQube with your CI/CD pipeline.
    - Configure SonarQube to analyze code for quality and security issues.

## $${\color {red} \textbf {Phase 3: CI/CD Setup}}$$

1. **Install Jenkins for Automation:**
    - Install Jenkins on the EC2 instance to automate deployment:
    Install Java
    
    ```bash
    sudo apt update
    sudo apt install fontconfig openjdk-17-jre
    java -version
    openjdk version "17.0.8" 2023-07-18
    OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
    OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)
    
    #jenkins
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
    echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
    ```
    
    - Access Jenkins in a web browser using the public IP of your EC2 instance.
        
        publicIp:8080
        
2. **Install Necessary Plugins in Jenkins:**

Goto Manage Jenkins →Plugins → Available Plugins →

Install below plugins

1 Eclipse Temurin Installer (Install without restart)

2 SonarQube Scanner (Install without restart)

3 NodeJs Plugin (Install Without restart)

4 Aws Credentials Plugin


### **Configure Java and Nodejs in Global Tool Configuration**

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save


### SonarQube

Create the token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

After adding sonar token

Click on Apply and Save

**The Configure System option** is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

**Install Docker Tools and Docker Plugins:**

- Go to "Dashboard" in your Jenkins web interface.
- Navigate to "Manage Jenkins" → "Manage Plugins."
- Click on the "Available" tab and search for "Docker."
- Check the following Docker-related plugins:
  - Docker
  - Docker Commons
  - Docker Pipeline
  - Docker API
  - docker-build-step
- Click on the "Install without restart" button to install these plugins.

**Add DockerHub Credentials:**

- To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
  - Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
  - Click on "System" and then "Global credentials (unrestricted)."
  - Click on "Add Credentials" on the left side.
  - Choose "Secret text" as the kind of credentials.
  - Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
  - Click "OK" to save your DockerHub credentials.


  **Add AWS Credentials:**

- To securely handle Aws credentials in your Jenkins pipeline, follow these steps:
  - Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
  - Click on "System" and then "Global credentials (unrestricted)."
  - Click on "Add Credentials" on the left side.
  - Choose "Secret text" as the kind of credentials.
  - Enter your Aws credentials (Access key and Secret Access Key) and give the credentials an ID (e.g., "aws").
  - Click "OK" to save your Aws credentials.


    Now, here you have to build the EKS pipeline code.


  ```groovy

 pipeline {
          agent any
          environment {
              // Set AWS credentials if using Jenkins credentials plugin
              AWS_ACCESS_KEY_ID = credentials('aws')
              AWS_SECRET_ACCESS_KEY = credentials('aws')
              AWS_DEFAULT_REGION = 'ap-south-1'  // Set your desired AWS region
          }
          parameters {
              choice(
                  name: 'action',
                  choices: ['apply', 'destroy'],
                  description: 'Choose the Terraform action to perform'
              )
          }
          stages {
              stage('Checkout from Git') {
                  steps {
                      echo "Checking out the code from Git..."
                      git branch: 'main', url: 'https://github.com/prathamnandgirwar/Hostar-clown.git'
                  }
              }
              
              stage('Check AWS Credentials') {
                  steps {
                      script {
                          echo "Checking AWS credentials..."
                          // Check AWS identity to verify credentials
                          sh 'aws sts get-caller-identity'
                      }
                  }
              }
              stage('Terraform version') {
                  steps {
                      echo "Checking Terraform version..."
                      sh 'terraform --version'
                  }
              }
      
              stage('Terraform init') {
                  steps {
                      echo "Initializing Terraform..."
                      dir('Terraform') {
                          sh 'terraform init'
                      }
                  }
              }
               stage('Terraform validate') {
                  steps {
                      echo "Validating Terraform configuration..."
                      dir('Terraform') {
                          sh 'terraform validate'
                      }
                  }
              }
      
              stage('Terraform plan') {
                  steps {
                      echo "Running Terraform plan..."
                      dir('Terraform') {
                          sh 'terraform plan'
                      }
                  }
              }
               stage('Terraform apply/destroy') {
                  steps {
                      echo "Applying or Destroying Terraform configuration..."
                      dir('Terraform') {
                          // Use the parameter to determine the action
                          sh "terraform ${params.action} --auto-approve"
                      }
                  }
              }
          }
          post {
              failure {
                  echo "Pipeline failed. Check the logs for errors."
              }
              success {
                  echo "Pipeline executed successfully!"
              }
          }
      }
```
             

Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub and AWS credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.


```groovy
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'nodejs16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/prathamnandgirwar/Youtube-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Youtube \
                    -Dsonar.projectKey=Youtube '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'p', usernameVariable: 'u')]) {
    // some block
                    sh "docker login -u ${env.u} -p ${env.p}"
                       sh "docker build --build-arg REACT_APP_RAPID_API_KEY=f5c3fe28c9msh069aa516d766c04p1969d1jsn085f99fefcd1 -t youtube ."
                       sh "docker tag youtube prathamnandgiwar/youtube:latest "
                       sh "docker push prathamnandgiwar/youtube:latest "
                    
                }
                 
                }
            
        }
        stage("TRIVY"){
            steps{
                sh "trivy image prathamnandgiwar/youtube:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name youtube -p 3000:3000 prathamnandgiwar/youtube:latest'
            }
        }
        
    }
}
```



                




          
            
            
       
