# Step 1: Launch AWS EC2 Instance

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

# Step 2: Install Git and Clone Repository

```bash
sudo yum install git -y

git clone https://github.com/surajyadav29/django-todo-ci-cd-jenkins-docker-aws.git

cd django-todo-ci-cd-jenkins-docker-aws
```

# Step 3: Check Python and Create Virtual Environment

```bash
python3.12 --version

python3.12 -m venv env

source env/bin/activate
```

# Step 4: Install Project Requirements

```bash
python -m pip install --upgrade pip

pip install -r requirements.txt
```

# Step 5: Verify Python and Django

```bash
python --version

python -m django --version
```

# Step 6: Configure Django Settings

Open settings file:

```bash
nano todoApp/settings.py
```

Update `ALLOWED_HOSTS` with your EC2 Public IPv4 address:

```python
ALLOWED_HOSTS = ['YOUR_PUBLIC_IP', 'localhost', '127.0.0.1']
```

Save: `Ctrl + O` → `Enter` → `Ctrl + X`.

# Step 7: Check Configuration and Apply Migrations

```bash
python manage.py check

python manage.py migrate
```

# Step 8: Run Django Application

```bash
python manage.py runserver 0.0.0.0:8000
```

Application URL:

```text
http://YOUR_PUBLIC_IP:8000/
```

# Step 9: Install Docker

```bash
sudo yum install docker -y

sudo systemctl start docker

sudo systemctl enable docker

docker --version
```

# Step 10: Create Dockerfile

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
```

# Step 11: Build Docker Image

```bash
sudo docker build . -t todo

sudo docker images
```

# Step 12: Run Docker Container

```bash
sudo docker run -d -p 8000:8000 --name todo-app todo

sudo docker ps
```

Application URL:

```text
http://YOUR_PUBLIC_IP:8000/
```

# Step 13: Install Jenkins Using Docker

```bash

sudo docker pull jenkins/jenkins:latest

sudo docker run -d -p 8080:8080 docker.io/jenkins/jenkins:latest

sudo docker ps

sudo docker logs <container-id>
```

Jenkins URL:

```text
http://YOUR_PUBLIC_IP:8080
```

# Step 14: Configure Jenkins Agent

# Unlock Jenkins
sudo docker exec [container-id] cat /var/jenkins_home/secrets/initialAdminPassword

Replace [container-id] with the Jenkins container ID.

Open Jenkins and paste the password into the Unlock Jenkins page.

Configure Jenkins Agent:

Get the current working directory:

pwd

```text
Name : todo-agent
Remote Root Directory : <output of pwd>
Labels : todo
```

# Step 15: Create Jenkins Build Job

- Source Code Management → Git
- Repository URL → GitHub repository
- Build Trigger → Build Now / GitHub Webhook
- Agent Label → todo

Build command:

```bash
sudo docker build . -t todo
```

Deployment command:

```bash
sudo docker run -d -p 8000:8000 todo
```

# Docker Maintenance Commands

```bash
# List containers
sudo docker ps

# Stop container
sudo docker stop <container-id>

# Remove container
sudo docker rm <container-id>

# View logs
sudo docker logs <container-id>

# List images
sudo docker images

# Remove image
sudo docker rmi <image-id>
```

## Author

Suraj Yadav

DevOps Engineer | AWS | Docker | Jenkins | CI/CD
