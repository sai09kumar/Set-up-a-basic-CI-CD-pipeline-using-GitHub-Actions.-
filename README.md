Step 1: Set Up AWS EC2 Instance
Launch an EC2 Instance

Go to AWS Management Console → EC2 → Launch Instance
Choose an Ubuntu 22.04 AMI
Select an instance type (e.g., t2.micro)
Configure security groups:
Allow SSH (port 22) → Your IP
Allow HTTP (port 80) → Public Access
Allow HTTPS (port 443) → Public Access
Add a key pair and download the .pem file
Launch the instance
Connect to EC2 via SSH

bash
Copy
Edit
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
Install Docker on EC2

bash
Copy
Edit
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
Step 2: Create a Web App and Dockerize It
Create a simple Node.js web app

bash
Copy
Edit
mkdir webapp && cd webapp
npm init -y
npm install express
Create index.js

javascript
Copy
Edit
const express = require("express");
const app = express();
app.get("/", (req, res) => res.send("🚀 Web App Running!"));
app.listen(3000, () => console.log("Server running on port 3000"));
Create a Dockerfile

dockerfile
Copy
Edit
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
Build and run the Docker container locally

bash
Copy
Edit
docker build -t mywebapp .
docker run -d -p 3000:3000 mywebapp
Step 3: Push Code to GitHub
Initialize Git

bash
Copy
Edit
git init
git add .
git commit -m "Initial commit"
git branch -M main
Create a GitHub Repo & Push Code

bash
Copy
Edit
git remote add origin https://github.com/your-username/webapp.git
git push -u origin main
Step 4: Set Up GitHub Actions for CI/CD
Go to GitHub → Your Repo → Actions

Click "New Workflow" and set up a workflow manually
Create .github/workflows/deploy.yml

yaml
Copy
Edit
name: Deploy Web App

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Build Docker Image
        run: |
          docker build -t your-dockerhub-username/webapp:latest .

      - name: Push Docker Image to Docker Hub
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push your-dockerhub-username/webapp:latest

      - name: Deploy to AWS EC2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull your-dockerhub-username/webapp:latest
            docker stop webapp || true
            docker rm webapp || true
            docker run -d -p 80:3000 --name webapp your-dockerhub-username/webapp:latest
Step 5: Store Secrets in GitHub
Go to GitHub → Repo → Settings → Secrets → Actions
Add the following secrets:
DOCKER_USERNAME → Your Docker Hub username
DOCKER_PASSWORD → Your Docker Hub password
EC2_HOST → Your EC2 Public IP
EC2_SSH_KEY → Contents of your your-key.pem
Step 6: Test the CI/CD Pipeline
Push Changes to GitHub

bash
Copy
Edit
git add .
git commit -m "Added GitHub Actions for CI/CD"
git push origin main
Check GitHub Actions

Go to GitHub → Actions
You should see the pipeline running
On success, your app is deployed!
Access the Web App

Open http://your-ec2-public-ip in your browser
You should see "🚀 Web App Running!"

Final Outcome 🚀
✅ Any push to main triggers GitHub Actions
✅ Builds & pushes a Docker image to Docker Hub
✅ SSHs into AWS EC2 & pulls the latest image
✅ Deploys and runs the web app

Your CI/CD pipeline is now fully automated! 
