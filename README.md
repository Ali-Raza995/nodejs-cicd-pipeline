<!DOCTYPE html>
<html>
<head>
</head>
<body>

<h1>Node.js Backend Deployment to EC2 with Docker, Docker Compose, and GitHub Actions</h1>

<p>This repository contains the configuration and workflows for deploying a Node.js backend to an EC2 instance using Docker, Docker Compose, and a CI/CD pipeline powered by GitHub Actions.</p>

<h2>Project Structure</h2>
<pre>
.
├── Dockerfile           # Defines the Docker image for the Node.js backend
├── docker-compose.yml   # Manages container orchestration (optional)
├── .github
│   └── workflows
│       └── deploy.yml   # GitHub Actions workflow for CI/CD pipeline
└── README.md            # Documentation
</pre>

<h2>1. Dockerfile</h2>
<p>The Dockerfile specifies the build instructions for the Node.js backend application.</p>
<pre>
<code>
# Use Node.js as the base image
FROM node:18

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the application code
COPY . .

# Expose the application port
EXPOSE 5000

# Start the application
CMD ["npm", "start"]
</code>
</pre>

<h2>2. Docker Compose</h2>
<p>Optional: Use Docker Compose for managing multiple services or simplified deployment.</p>
<pre>
<code>
version: "3.8"
services:
  node-backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    restart: always
</code>
</pre>

<h2>3. GitHub Actions CI/CD Pipeline</h2>
<p>The CI/CD pipeline automates the process of building, pushing, and deploying the Docker image to the EC2 instance.</p>

<h3>GitHub Actions Workflow: <code>.github/workflows/deploy.yml</code></h3>
<pre>
<code>
name: Deploy Node.js Backend to EC2 with Docker

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: |
          IMAGE_NAME="${{ secrets.DOCKER_USERNAME }}/node-backend:latest"
          docker build -t $IMAGE_NAME .

      - name: Push Docker image to Docker Hub
        run: |
          IMAGE_NAME="${{ secrets.DOCKER_USERNAME }}/node-backend:latest"
          docker push $IMAGE_NAME

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: SSH into EC2 and deploy Docker image
        uses: appleboy/ssh-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          port: 22
          script: |
            IMAGE_NAME="${{ secrets.DOCKER_USERNAME }}/node-backend:latest"
            echo "Docker image to be pulled: $IMAGE_NAME"
            sudo docker stop node-backend || true
            sudo docker rm node-backend || true
            sudo docker pull $IMAGE_NAME
            sudo docker run -d --name node-backend -p 5000:5000 $IMAGE_NAME
</code>
</pre>

<h2>4. Secrets Configuration</h2>
<p>Configure the following secrets in your GitHub repository:</p>
<ul>
  <li><code>DOCKER_USERNAME</code>: Your Docker Hub username</li>
  <li><code>DOCKER_PASSWORD</code>: Your Docker Hub password</li>
  <li><code>EC2_HOST</code>: Public IP or domain of your EC2 instance</li>
  <li><code>EC2_SSH_KEY</code>: Private SSH key for accessing the EC2 instance</li>
</ul>

<h2>5. Deployment Steps</h2>
<ol>
  <li>Push your code to the <code>main</code> branch.</li>
  <li>GitHub Actions will automatically:
    <ul>
      <li>Build the Docker image</li>
      <li>Push the Docker image to Docker Hub</li>
      <li>SSH into the EC2 instance and deploy the updated image</li>
    </ul>
  </li>
</ol>

<h2>6. Accessing the Application</h2>
<p>Once the deployment is complete, the application will be accessible at:</p>
<pre>
http://<your-ec2-public-ip>:5000
</pre>

<h2>7. Best Practices</h2>
<ul>
  <li>Use environment variables to manage sensitive data.</li>
  <li>Rotate keys and passwords regularly.</li>
  <li>Monitor GitHub Actions logs for successful deployments.</li>
</ul>

</body>
</html>
