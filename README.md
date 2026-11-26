# Django Todo CI/CD Deployment Using AWS EC2, Docker and Jenkins

A hands-on DevOps project demonstrating automated deployment of a Django Todo application using **AWS EC2, Docker, Jenkins, and GitHub**.

## Project Overview

This project automates the deployment of a Django Todo application using Jenkins CI/CD.

Whenever code is updated in GitHub, Jenkins can execute the deployment process on an EC2 Jenkins agent, build a Docker image, and deploy the updated application.

## Technologies Used

- AWS EC2
- Amazon Linux
- Git and GitHub
- Python 3.12
- Django
- Docker
- Jenkins
- Linux

## Architecture

```text
Developer
    |
    v
GitHub Repository
    |
    v
Jenkins Controller (EC2)
    |
    v
Jenkins Agent (EC2)
    |
    v
Docker Build
    |
    v
Docker Container
    |
    v
Django Todo Application
    |
    v
Browser (Port 8000)
```

## Step 1: Launch AWS EC2 Instances

Launch two Amazon Linux EC2 instances:

1. Jenkins Controller
2. Jenkins Agent / Application Server

Configure the required Security Group rules.

| Port | Purpose |
|---|---|
| 22 | SSH access |
| 8000 | Django application |
| 8080 | Jenkins web interface |

Restrict access to these ports to trusted IP addresses or security groups wherever possible.

Connect to EC2:

```bash
ssh -i "your-key.pem" ec2-user@YOUR_PUBLIC_IP
```

## Step 2: Install Git and Clone Repository

Install Git:

```bash
sudo yum install -y git
```

Clone the repository:

```bash
git clone https://github.com/surajyadav29/django-todo-ci-cd-jenkins-docker-aws.git
```

Go to the project directory:

```bash
cd django-todo-ci-cd-jenkins-docker-aws
```

## Step 3: Install Python 3.12

Install Python:

```bash
sudo yum install -y python3.12 python3.12-pip
```

Check the version:

```bash
python3.12 --version
```

Create a virtual environment:

```bash
python3.12 -m venv venv
```

Activate it:

```bash
source venv/bin/activate
```

## Step 4: Install Project Requirements

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

Install project dependencies:

```bash
pip install -r requirements.txt
```

Check Django version:

```bash
python -m django --version
```

## Step 5: Configure Django Settings

Open the settings file:

```bash
nano todoApp/settings.py
```

Update `ALLOWED_HOSTS`:

```python
ALLOWED_HOSTS = ['YOUR_PUBLIC_IP', 'localhost', '127.0.0.1']
```

Save the file.

## Step 6: Check Django and Apply Migrations

Check the Django configuration:

```bash
python manage.py check
```

Apply database migrations:

```bash
python manage.py migrate
```

## Step 7: Create Dockerfile

Create the Dockerfile:

```bash
nano Dockerfile
```

Add the following configuration:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### Dockerfile Explanation

| Instruction | Purpose |
|---|---|
| FROM | Selects the Python base image |
| WORKDIR | Sets the working directory |
| COPY | Copies project files |
| RUN | Executes commands during image creation |
| EXPOSE | Documents the application port |
| CMD | Starts the Django application |

## Step 8: Create .dockerignore

Create the `.dockerignore` file:

```bash
nano .dockerignore
```

Add:

```text
venv/
.venv/
__pycache__/
*.pyc
.git/
.gitignore
.env
*.sqlite3
.vscode/
.idea/
```

This prevents unnecessary files from being included in the Docker build context.

## Step 9: Build Docker Image

Install Docker if it is not already installed:

```bash
sudo yum install -y docker
```

Start Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Build the image:

```bash
sudo docker build -t todo .
```

Check Docker images:

```bash
sudo docker images
```

## Step 10: Run Docker Container

Run the application:

```bash
sudo docker run -d -p 8000:8000 --name todo-app todo
```

Check running containers:

```bash
sudo docker ps
```

Access the application:

```text
http://YOUR_PUBLIC_IP:8000/todos/
```

Ensure Django allows the EC2 public IP in `ALLOWED_HOSTS`.

Stop the directly running Django development server before using the same host port for Docker.

## Step 11: Install Jenkins on Controller EC2 Using Docker

Install Java 21:

```bash
sudo yum install java-21-amazon-corretto-headless -y
```

Verify Java:

```bash
java -version
```

Pull the Jenkins image:

```bash
sudo docker pull jenkins/jenkins:latest
```

Run Jenkins with persistent storage:

```bash
sudo docker run -d -p 8080:8080 docker.io/jenkins/jenkins:latest
```

Check the container:

```bash
sudo docker ps
```

Get the initial administrator password:

```bash
sudo docker exec (container id) cat /var/jenkins_home/secrets/initialAdminPassword
```

Open Jenkins:

```text
http://YOUR_CONTROLLER_PUBLIC_IP:8080
```

Unlock Jenkins, install the suggested plugins, and create your administrator account.

## Step 12: Configure Jenkins Agent

In Jenkins, navigate to:

```text
Manage Jenkins → Nodes → New Node
```

Configure the agent:

| Setting | Value |
|---|---|
| Name | todo-agent |
| Remote Root Directory | /home/ec2-user |
| Labels | todo |
| Launch method | Launch agent by connecting it to the controller |

Enable WebSocket connection if available.

Save the configuration and open the agent's Status page.

Download the agent JAR on the Agent EC2:

```bash
cd /home/ec2-user

curl -sO http://YOUR_CONTROLLER_PUBLIC_IP:8080/jnlpJars/agent.jar
```

Connect the agent using the newly generated secret from the Jenkins agent Status page at detach mode:

```bash
nohup java -jar agent.jar \
-url http://YOUR_CONTROLLER_PUBLIC_IP:8080/ \
-secret YOUR_NEW_AGENT_SECRET \
-name todo-agent \
-webSocket \
-workDir /home/ec2-user \
> jenkins-agent.log 2>&1 &
```

Keep the agent process running while Jenkins jobs are using it.

## Step 13: Create Jenkins Build Job

1. Open Jenkins.
2. Select **New Item**.
3. Enter the job name: `django-todo-deployment`.
4. Select **Freestyle project**.
5. Open Configure.

```
6. Under Build Steps, select **Execute shell**.

Add the deployment script:

```bash
cd /home/ec2-user/django-todo-ci-cd-jenkins-docker-aws
sudo docker build -t todo .
sudo docker stop todo-app || true
sudo docker rm todo-app || true
sudo docker run -d -p 8000:8000 --name todo-app todo
sudo docker exec todo-app python manage.py migrate
```

```bash
sudo docker ps
```

Save the job.

Click **Build Now** to execute the deployment.

## Step 14: Verify Deployment

Check the Jenkins build status:

```text
Jenkins → Your Job → Build History → Console Output
```

Expected result:

```text
Finished: SUCCESS
```

Check Docker containers on the Agent EC2:

```bash
sudo docker ps
```

Open the application:

```text
http://YOUR_AGENT_PUBLIC_IP:8000/todos/
```

# After any changes at project

sudo docker ps -a

sudo docker kill (container id)

Jenkins - Build Now ( It will run the container and do changes)

OR

## Step 15: GitHub Commands

Check modified files:

```bash
git status
```

Add changes:

```bash
git add .
```

Commit:

```bash
git commit -m "Update Django application and Docker configuration"
```

Push:

```bash
git push origin main
```

Jenkins - Build now 
Changes will be done 

Keep the `venv/` directory excluded using `.gitignore`.

Example `.gitignore` entries:

```gitignore
venv/
.venv/
__pycache__/
*.pyc
.env
*.sqlite3
.vscode/
.idea/
```

Use a GitHub Personal Access Token when Git requests an HTTPS password. Never commit tokens or Jenkins agent secrets.

## Step 16: Disconnect Jenkins Agent

Find the agent process:

```bash
ps -ef | grep '[a]gent.jar'
```

Stop it using its actual process ID:

```bash
kill YOUR_AGENT_PID
```

Verify:

```bash
ps -ef | grep '[a]gent.jar'
```

## Important Notes

- Jenkins Controller and Jenkins Agent can run on separate EC2 instances.
- The Jenkins Agent needs Docker installed and permission to execute Docker commands.
- The agent must be connected for Jenkins to deploy through that agent.
- Docker containers continue running independently after a Jenkins job is deleted.
- The SQLite database inside a container can be lost when the container is removed. Persistent storage is needed to retain application data.
- Django's `runserver` is for development and demonstration, not production deployment. A production setup should use a production application server such as Gunicorn and an appropriate web server.
- Restrict SSH, Jenkins and application ports to trusted sources.

## Project Author

**Suraj Yadav**

DevOps Engineer | AWS | Docker | Jenkins | CI/CD
