###  AWS CI/CD Pipeline with EC2, Docker, and GitHub Actions 

This project demonstrates setting up a **CI/CD pipeline** for a **Dockerized Node.js application** using **GitHub Actions and AWS EC2**.  
The pipeline automates:  
-  **Building a Docker image** and pushing it to **Docker Hub**.  
 - **Deploying the container** to an **EC2 instance** using a **self-hosted runner**.  
 - **Setting up Nginx** as a reverse proxy for handling incoming requests.  

  


##  Setting Up the Project 

 Prerequisites 
 **Docker Installed** on your local machine.  
 **AWS Account** with EC2 access.  
 **GitHub Repository** with Actions enabled.  
 **Docker Hub Account** to store images.  
![image](https://github.com/user-attachments/assets/a657b8ea-026b-4438-8897-37bd3656e0a9)


##  Steps

### **1️⃣ Setting Up the Node.js Application with Docker**  

  Create a Node.js App 
```sh
mkdir my-node-app && cd my-node-app
npm init -y
npm install express
```

 Write `server.js` and Initialize Git 
```sh
git init
```

  Create a Dockerfile 
```sh
docker init
```

  Push to GitHub & Enable GitHub Actions 
1. Go to your **GitHub repo → Actions**  
2. Select **Docker Image CI workflow** to automate Docker builds.  

![image](https://github.com/user-attachments/assets/8a42440f-7f5f-4e56-8663-d17e02cc2103)

### **2️⃣ Configuring the CI Pipeline (GitHub Actions)**  

 Setup `.github/workflows/ci.yml` 
```yaml
name: Docker CI Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Build Docker image
        run: docker build -t yourdockerhubusername/aws_docker_cicd .

      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Push Docker image to Docker Hub
        run: docker push yourdockerhubusername/aws_docker_cicd:latest
```

  Store Docker Hub Credentials in GitHub 
1. **Go to your GitHub Repo** → Settings → Secrets & Variables → Actions.  
2. Add **DOCKER_USERNAME** and **DOCKER_PASSWORD**.  

 Create a Docker Hub Repo 
Make sure the **Docker Hub repository name** matches the one in `ci.yml`.  

![image](https://github.com/user-attachments/assets/f6d54ec4-e6ee-4971-84fd-27493da1aa10)

---
### **3️⃣ Setting Up an AWS EC2 Instance**  

 Launch an EC2 Instance 
1. **Create an EC2 instance** with **Ubuntu (Free Tier)**.  
2. **Set up a Security Group** to allow **all traffic**.  
3. **Create a key pair** for SSH access.  
4. **Launch the instance**.

  Connect to EC2 via SSH 
 
---

### **4️⃣ Configuring a Self-Hosted GitHub Runner on EC2**  

1. **Go to GitHub Actions → Settings** → Actions → Runners → Self-hosted.  
2. Copy the provided command and paste it into the EC2 terminal.  

```sh
cd ~/actions-runner
./config.sh --url https://github.com/yourrepo --token YOUR_GITHUB_TOKEN
./run.sh
```

---

### **5️⃣ Setting Up the CD Pipeline (GitHub Actions)**  

 Configure `.github/workflows/cd.yml` 
```yaml
name: Docker CD Pipeline

on:
  workflow_run:
    workflows: ["Docker CI Pipeline"]
    types:
      - completed

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Pull Docker image
        run: sudo docker pull yourdockerhubusername/aws_docker_cicd:latest

      - name: Delete Old Docker Container
        run: sudo docker rm -f aws_docker_cicd-container || true

      - name: Run Docker Container
        run: sudo docker run -d -p 8080:8080 --name aws_docker_cicd-container yourdockerhubusername/aws_docker_cicd
```

 This pipeline automatically **pulls, removes old containers, and deploys the latest version**.  

---

### **6️⃣ Installing Nginx on EC2 (Reverse Proxy Setup)**  

  Install Nginx 
```sh
sudo apt update
sudo apt install nginx -y
```

 Find Docker Container IP Address 
 this is the method i use:
```sh
sudo docker exec -it aws_docker_cicd-container ifconfig
```

  Configure Nginx as a Reverse Proxy 
Edit the Nginx configuration file:  
```sh
cd /etc/nginx/sites-available/
sudo nano default
```

Add the following inside the `server` block:  
```nginx
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://172.17.0.2:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

 Restart Nginx 
```sh
sudo systemctl restart nginx
```

 Now, your EC2 instance routes traffic from **port 80 to your Docker container (8080)**.  

---

### **7️⃣ Testing the Deployment**  

  Check if the Container is Running 
```sh
sudo docker ps
```

  Access the Application 
Get your **EC2 Public IPv4 DNS** from AWS:  
```sh
http://ec2-your-public-ip.compute-1.amazonaws.com
```
 
---

## **📜 Credit**  
Inspired by [this tutorial](https://www.youtube.com/watch?v=rRes9LM-Jh8).  
 
