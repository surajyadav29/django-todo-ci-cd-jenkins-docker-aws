# Django Todo CI/CD Deployment Using AWS EC2, Docker and Jenkins

## Step 1: Launch AWS EC2 Instance

- Launch an Amazon Linux EC2 instance.
- Create/select a key pair.
- Configure Security Group inbound rules:

| Port | Purpose |
|---|---|
| 22 | SSH |
| 8000 | Django Application |
| 8080 | Jenkins |

Connect to EC2:

```bash
ssh -i "your-key.pem" ec2-user@YOUR_PUBLIC_IP
```

## Step 2: Install Git and Clone Repository

```bash
sudo yum install -y git

git clone https://github.com/surajyadav29/django-todo-ci-cd-jenkins-docker-aws.git

cd django-todo-ci-cd-jenkins-docker-aws
```

## Step 3: Install Python 3.12 and Create Virtual Environment

Install Python 3.12 and pip:

```bash
sudo dnf install python3.12 python3.12-pip -y
```

**Understanding Python and pip:**

**1. `python3.12` – Python interpreter**

Meaning: Python interpreter is software that understands and executes Python code.

Example:

```bash
python3.12 --version
```

**2. `python3.12-pip` – pip package manager**

Meaning: pip is a tool used to install Python libraries and packages.

Example:

```bash
pip install django
```

**Difference between Python and pip:**

| Python 3.12 | pip |
|---|---|
| Executes Python code | Installs Python packages |
| Runs Django application | Installs Django |
| Runs manage.py | Installs project requirements |
| Works as an interpreter | Works as a package manager |

Check Python version:

```bash
python3.12 --version
```

Create virtual environment:

```bash
python3.12 -m venv venv
```

Activate virtual environment:

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

## Step 5: Verify Python and Django

Check Python version:

```bash
python --version
```

Check Django version:

```bash
python -m django --version
```

## Step 6: Configure Django Settings

Open settings file:

```bash
nano todoApp/settings.py
```

Update `ALLOWED_HOSTS` with your EC2 Public IPv4 address:

```python
ALLOWED_HOSTS = ['YOUR_PUBLIC_IP', 'localhost', '127.0.0.1']
```

Save: `Ctrl + O` → `Enter` → `Ctrl + X`.

## Step 7: Check Configuration and Apply Migrations

Check Django configuration:

```bash
python manage.py check
```

Apply database migrations:

```bash
python manage.py migrate
```

## Step 8: Run Django Application

```bash
python manage.py runserver 0.0.0.0:8000
```

Application URL:

```text
http://YOUR_PUBLIC_IP:8000/
```

## Step 9: Install Docker

```bash
sudo yum install docker -y

sudo systemctl start docker

sudo systemctl enable docker

docker --version
```

## Step 10: Create Dockerfile

```bash
vi Dockerfile
```

Add:

```dockerfile
FROM python:3.12

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]


This means install all Python packages listed in the requirements.txt file.
Simple explanation
- pip – Python package installer.
- install – Installs packages.
- -r – Reads a requirements file.
- requirements.txt – Contains the names and versions of Python packages required by your Django project.
Example
Suppose your requirements.txt contains:
Django==6.0.5
requests==2.32.3
gunicorn==23.0.0

```

## Step 11: Build Docker Image

```bash
sudo docker build . -t todo

sudo docker images
```

## Step 12: Run Docker Container

```bash
sudo docker run -d -p 8000:8000 --name todo-app todo

sudo docker ps
```

Application URL:

```text
http://YOUR_PUBLIC_IP:8000/
```

## Step 13: Install Jenkins Using Docker

Pull Jenkins image:

```bash
sudo docker pull jenkins/jenkins:latest
```

Run Jenkins container:

```bash
sudo docker run -d -p 8080:8080 docker.io/jenkins/jenkins:latest
```

Check running containers:

```bash
sudo docker ps
```

View Jenkins logs:

```bash
sudo docker logs <container-id>
```

Jenkins URL:

```text
http://YOUR_PUBLIC_IP:8080
```

## Step 14: Configure Jenkins Agent

### Unlock Jenkins

Get the initial administrator password:

```bash
sudo docker exec <container-id> cat /var/jenkins_home/secrets/initialAdminPassword
```

Replace `<container-id>` with the Jenkins container ID.

Open Jenkins and paste the password into the Unlock Jenkins page.

### Get the current working directory

```bash
pwd
```

### Configure Jenkins Agent

```text
Name : todo-agent

Remote Root Directory : <output of pwd>

Labels : todo
```

## Step 15: Create Jenkins Build Job

Configure the following:

- Source Code Management → Git
- Repository URL → GitHub repository
- Build Trigger → Build Now / GitHub Webhook
- Agent Label → todo

### Build command

```bash
sudo docker build . -t todo
```

### Deployment command

```bash
sudo docker run -d -p 8000:8000 todo
```

## Docker Maintenance Commands

### List containers

```bash
sudo docker ps
```

### Stop container

```bash
sudo docker stop <container-id>
```

### Remove container

```bash
sudo docker rm <container-id>
```

### View logs

```bash
sudo docker logs <container-id>
```

### List images

```bash
sudo docker images
```

### Remove image

```bash
sudo docker rmi <image-id>
```

## Author

**Suraj Yadav**

DevOps Engineer | AWS | Docker | Jenkins | CI/CD
