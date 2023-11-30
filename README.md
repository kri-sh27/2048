# Getting Started with 2048 | CI/CD Pipeline Automation



This game (2048) was built using **React** and **TypeScript**. The unique part of this example is animations. The animations in React aren't that straightforward, so I hope you can learn something new from it.



<!-- ## How To Play? -->

<!-- You can play 2048 on [Github pages](https://mateuszsokola.github.io/2048-in-react/) -->

## Available Scripts

In the project directory, you can run:

### `yarn start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

### `yarn build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).



### DevSecOps: Deploying the 2048 Game on Docker and with Jenkins CI/CD

Steps:-

Step 1 — Launch an Ubuntu(22.04) T2 Large Instance

Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.

Step 3 — Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check.

Step 4 — Create a Pipeline Project in Jenkins using a Declarative Pipeline

Step 5 — Install OWASP Dependency Check Plugins

Step 6 — Docker Image Build and Push

Step 7 — Deploy the image using Docker

Step 9 — Access the Game on Browser.

Step 10 — Terminate the AWS EC2 Instances.

# STEP1:Launch an Ubuntu(22.04) T2 Large Instance
 Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group and open all ports (not best case to open all ports but just for learning purposes it's okay).

# Step 2 — Install Jenkins, Docker and Trivy
 Connect to your console, and enter these commands to Install Jenkins

 ```bash
 vi jenkins.sh
```
```bash
#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```
```bash
sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins
```

Once Jenkins is installed, you will need to go to your AWS EC2 Security Group and open Inbound Port 8080, since Jenkins works on Port 8080.

Now, grab your Public IP Address

```bash
<EC2 Public IP Address:8080>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Unlock Jenkins using an administrative password and install the suggested plugins.

Jenkins will now get installed and install all the libraries.

Create a user click on save and continue.

Jenkins Getting Started Screen.

# 2B — Install Docker
```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   #my case is ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
```
After the docker installation, we create a sonarqube container (Remember to add 9000 ports in the security group).

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
Now our sonarqube is up and running.

Enter username and password, click on login and change password

```bash
username admin
password admin
```
Update New password, This is Sonar Dashboard.
<div align="center">
  <img src="./Output/3.png" alt="Logo" width="100%" height="100%">
</div>

<br />

# 2C — Install Trivy

```bash 
vi trivy.sh
```

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update

sudo apt-get install trivy -y
```

Next, we will log in to Jenkins and start to configure our Pipeline in Jenkins

# Step 3 — Install Plugins like JDK, Sonarqube Scanner, NodeJs, OWASP Dependency Check

##  3A — Install Plugin

Install below plugins

1 → Eclipse Temurin Installer (Install without restart)

2 → SonarQube Scanner (Install without restart)

3 → NodeJs Plugin (Install Without restart)

## 3B — Configure Java and Nodejs in Global Tool Configuration

Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save
<div align="center">
  <img src="./Output/1.png" alt="Logo" width="100%" height="100%">
</div>

<br />
<div align="center">
  <img src="./Output/2.png" alt="Logo" width="100%" height="100%">
</div>

<br />
## 3C — Create a Job

create a job as 2048 Name, select pipeline and click on ok.

# Step 4 — Configure Sonar Server in Manage Jenkins
Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000, so <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

click on update Token

Create a token with a name and generate

copy Token

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this


Now, go to Dashboard → Manage Jenkins → System and Add like the below image.

Click on Apply and Save

The Configure System option is used in Jenkins to configure different server

Global Tool Configuration is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

<div align="center">
  <img src="./Output/4.png" alt="Logo" width="100%" height="100%">
</div>

<br />

In the Sonarqube Dashboard add a quality gate also

Administration--> Configuration-->Webhooks

Add details
<div align="center">
  <img src="./Output/5.png" alt="Logo" width="100%" height="100%">
</div>

<br />

```bash
#in url section of quality gate
<http://jenkins-public-ip:8080>/sonarqube-webhook/
```

# Let's create Pipeline and add the script in our Pipeline Script.

```bash
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
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
                git branch: 'main', url: 'https://github.com/kri-sh27/2048.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}
```

Click on Build now, you will see the stage view like this

<div align="center">
  <img src="./Output/Untitled.png" alt="Logo" width="100%" height="100%">
</div>

<br />



To see the report, you can go to Sonarqube Server and go to Projects.

<div align="center">
  <img src="./Output/8.png" alt="Logo" width="100%" height="100%">
</div>

<br />

You can see the report has been generated and the status shows as passed. You can see that there are 838 lines. To see a detailed report, you can go to issues.

# Step 5 — Install OWASP Dependency Check Plugins

GotoDashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.

First, we configured the Plugin and next, we had to configure the Tool

Goto Dashboard → Manage Jenkins → Tools →

<div align="center">
  <img src="./Output/10.png" alt="Logo" width="100%" height="100%">
</div>

<br />



Click on Apply and Save here.

Now go configure → Pipeline and add this stage to your pipeline and build.

```bash
stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
```
# Step 6 — Docker Image Build and Push
we need to install the Docker tool in our system, Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins

Docker

Docker Commons

Docker Pipeline

Docker API

docker-build-step

and click on install without restart

Now, goto Dashboard → Manage Jenkins → Tools →

<div align="center">
  <img src="./Output/11.png" alt="Logo" width="100%" height="100%">
</div>

<br />


Add this stage to Pipeline Script

```bash
stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t 2048 ."
                       sh "docker tag 2048 krishnahogale/2048:latest "
                       sh "docker push krishnahogale/2048:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image krishnahogale/2048:latest > trivy.txt" 
            }
        }
```

When you log in to Dockerhub, you will see a new image is created

Now Run the container to see if the game coming up or not by adding below stage

```bash
stage('Deploy to container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 krishnahogale/2048:latest'
            }
        }
```


```bash
<Jenkins-public-ip:3000>
```

You will get this output

<div align="center">
  <img src="./Output/15.png" alt="Logo" width="100%" height="100%">
</div>

<br />

