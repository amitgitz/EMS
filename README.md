# Django Employee Management System
Employee Management System (EMS) EC2 Instance Deployment using Docker, Docker-Compose, Jenkins, Webhook and Slack Notification.

Setup on EC2 using Docker, Docker Compose, Jenkins, Webhook, and Slack.
## Step 1: Create an EC2 Instance with the following Configuration
1. Name: EMS
2. AMI: Ubuntu 20
3. Size: t2.medium
4. Security Group (SG):
  SSH - MyIP
    - 8000 - Anywhere IPV6
    - 8000 - Anywhere IPV4
    - 8080 - Anywhere IPV4
In the Advanced section of the instance, execute the following script to set up the Jenkins server:

``` bash
#!/bin/bash
sudo apt update
sudo apt install openjdk-11-jdk -y
sudo apt install openjdk-8-jdk -y
sudo apt install maven -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y
###
```

## Step 2: Jenkins Setup and Configuration
Launch the EC2 instance and wait for it to start. After some time, access your instance at publicipaddress:8080 to access Jenkins. Follow these steps to configure Jenkins:

Connect to the instance using SSH: `cat /var/lib/jenkins/secrets/initialAdminPassword`
Copy the Jenkins directory address and retrieve the initial admin password.

1. Set up Jenkins with default configuration.

2. Go to "Manage Jenkins" -> "Manage Plugins" -> "Available Plugins" and install the "Slack Notification" 
3. Install plugin without starting it.

## Step 3: GitHub Configuration
 - Go to your GitHub account settings.
 - Navigate to "Developer settings" -> "Personal access tokens" -> "Tokens (classic)" -> "Generate new token".
 - Provide a note name, select the appropriate repository access, and create the token.
 - Copy the generated token and store it in a safe place.

## Step 4: Slack Configuration
- Sign up for Slack using your email and create a workspace (EMP-Management).
- Create a channel named "#jenkinscicd" in the workspace.
- Search for "Jenkins CI" integration on Google and follow the instructions to integrate Jenkins with your Slack workspace and the newly created channel (#jenkinscicd).
- Copy the Team subdomain (e.g., empmanagement) and the Integration token credential ID (e.g., mIwoxsyzprerssdd) and store them in a safe place.

## Step 5: Jenkins Configure System
- Go back to Jenkins and navigate to "Manage Jenkins" -> "Configure System" -> "GitHub".
Configure the GitHub connection:
- Name: GitHub-Connection
- API URL: Default
- Add Credential:
- Kind: Secret text
    - Secret: GitHub personal access token
    - ID: Git-Connection
    - Description: Git-Connection
 Configure the Slack integration:
  - Workspace: EMP-Management
  - Credentials:
        - Kind: Secret text
        - Secret: Slack credential ID
Test the connection and save the configuration.`

## Step 6: Making our EMS Web Live
SSH into the EC2 instance using Bash and navigate to the Ubuntu directory:

```bash
$ mkdir projects
$ cd projects
$ git clone https://github.com/amitgitz/EMS.git ```
Navigate to the project folder:

```bash
$ cd EMS ```
**Install the required dependencies:**
```bash
$ sudo apt update
$ sudo apt install python3-pip
$ pip3 install django
```
**Launch the website with the following commands:**

```bash
$ python manage.py makemigrations
$ python manage.py migrate
$ python manage.py runserver 0.0.0.0:8000 ```

Access the website using the EC2 instance's public IP address and port 8000.

## Step 7: Installing Docker & Docker Compose
Install Docker and Docker Compose:

```bash
$ sudo apt update
$ sudo apt install docker.io
$ sudo apt install docker-compose
```
**Create the Dockerfile in the EMS directory:**
```bash
$ vi Dockerfile ```

**Add the followingcontent to the Dockerfile:**
`Dockerfile`

```bash
FROM python:3
RUN pip install django==3.2

COPY . .

RUN python manage.py migrate

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

```
**Create the Docker image and run the application: **
```bash

$ sudo docker build . -t ems:v1
$ sudo docker run -p 8000:8000 ems:v1
```
**Access the website using the EC2 instance's public IP address and port 8000**.

Stop the Docker container using the following commands:
```bash
$ sudo docker ps
$ sudo docker kill <PID>
```
Create the Docker Compose file:
```bash
$ vi docker-compose.yml
```
**Add the following content to the `docker-compose.yml` file:**
``` 
version: "3.3"
services:
  web:
    build: .
    ports:
      - "8000:8000"
      ```
      
Save the `docker-compose.yml` file and push the code to the GitHub repository:
```bash
$ git add .
$ git commit -m "Adding Docker and Docker Compose files"
$ git push origin jenkins-cicd
```

## Step 8: Webhook Configuration
In the GitHub repository settings, navigate to `"Webhooks"` and add a new webhook:

1. Payload URL: http://EC2-ipaddress:8080/github-webhook/
2. Content type: application/json
3. Events: "Just the push event"
Click on `"Create webhook"`

## Step 9: Jenkins Build
Go back to Jenkins and click on "Build Now". Check the application at `http://EC2-ipaddress:8000.`

Congratulations! Your application is deployed, and you should receive notifications in the Slack channel indicating the success of the build.

 - Now, any changes made to the web app will be automatically deployed without rebuilding using the webhook.
 - If you have any questions or encounter any problems with this project, feel free to connect with me on https://www.linkedin.com/in/devopsamit/
