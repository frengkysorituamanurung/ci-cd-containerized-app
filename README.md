# CI/CD Pipeline with Docker, GitHub Actions, and Deployment to DigitalOcean

This repository demonstrates a complete CI/CD pipeline using containerization with Docker and GitHub Actions. It automates the process of building a Docker image, pushing it to Docker Hub, and deploying it to a DigitalOcean droplet. The application used is a simple example, and the focus is on the CI/CD pipeline implementation.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
  - [1. GitHub Secrets](#1-github-secrets)
  - [2. DigitalOcean Droplet Setup](#2-digitalocean-droplet-setup)
  - [3. Docker Hub Setup](#3-docker-hub-setup)
- [CI/CD Pipeline Workflow](#cicd-pipeline-workflow)
- [Testing the Pipeline](#testing-the-pipeline)
- [Bonus: Deployment to DigitalOcean](#bonus-deployment-to-digitalocean)
- [Conclusion](#conclusion)

## Overview

This project implements a CI/CD pipeline that automates:
1. Building a Docker image from the source code.
2. Pushing the Docker image to Docker Hub.
3. Deploying the Docker container to a DigitalOcean droplet.

## Prerequisites

Before starting, ensure that you have the following:

- A GitHub account.
- A Docker Hub account to store Docker images.
- A DigitalOcean account and an active droplet with SSH access.
- An SSH key pair for authentication with the droplet.

## Setup

### 1. GitHub Secrets

You need to set up GitHub Secrets to securely store sensitive data. Go to your GitHub repository’s **Settings** > **Secrets and variables** > **Actions** > **New repository secret** and add the following:

- **`SSH_PRIVATE_KEY`**: The private SSH key to access the DigitalOcean droplet. Ensure it's in PEM format without a passphrase.
- **`DROPLET_IP`**: The public IP address of your DigitalOcean droplet.
- **`DOCKER_USERNAME`**: Your Docker Hub username.
- **`DOCKER_PASSWORD`**: Your Docker Hub password.

### 2. DigitalOcean Droplet Setup

1. **Create a Droplet**:
   - Set up a new droplet from the DigitalOcean dashboard and ensure Docker is installed. You can select an image with Docker pre-installed or manually install Docker using:
     ```bash
     sudo apt update
     sudo apt install docker.io
     ```

2. **Add the Public SSH Key**:
   - Add the SSH public key (from the key pair that matches `SSH_PRIVATE_KEY`) to the `~/.ssh/authorized_keys` file on the droplet:
     ```bash
     echo "your-public-key" >> ~/.ssh/authorized_keys
     ```
   - Ensure correct permissions:
     ```bash
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/authorized_keys
     ```

3. **Ensure Docker is Running**:
   - Verify that Docker is running with:
     ```bash
     sudo systemctl start docker
     sudo systemctl enable docker
     ```

### 3. Docker Hub Setup

1. **Create a Repository**:
   - Log in to Docker Hub and create a new public or private repository (e.g., `mydockerhubuser/my-app`).

2. **Authenticate Docker in the Workflow**:
   - The GitHub Actions workflow will log in to Docker Hub and push the built image to this repository.

## CI/CD Pipeline Workflow

The CI/CD pipeline is managed through GitHub Actions and is defined in the `.github/workflows/main.yml` file. The pipeline will trigger automatically on any push to the `main` branch.

### Workflow File (`.github/workflows/main.yml`):

```yaml
name: CI/CD Pipeline

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

    - name: Log in to Docker Hub
      run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

    - name: Build Docker image
      run: docker build . -t my-app:latest

    - name: Push Docker image to Docker Hub
      run: |
        docker tag my-app:latest mydockerhubuser/my-app:latest
        docker push mydockerhubuser/my-app:latest

    - name: Set up SSH Key
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa

    - name: Add DigitalOcean droplet to known hosts
      run: ssh-keyscan -H ${{ secrets.DROPLET_IP }} >> ~/.ssh/known_hosts

    - name: Deploy to DigitalOcean Droplet
      if: github.ref == 'refs/heads/main'
      run: ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no root@${{ secrets.DROPLET_IP }} '
        docker pull mydockerhubuser/my-app:latest &&
        docker stop my-app || true &&
        docker rm my-app || true &&
        docker run -d -p 80:3000 --name my-app mydockerhubuser/my-app:latest
      '
```

## Testing the Pipeline

1. **Push Changes**: 
   Once the workflow is configured, any push to the `main` branch will trigger the CI/CD pipeline.

2. **Monitor Workflow**: 
   Go to the **Actions** tab in your GitHub repository to monitor the progress of the workflow.

3. **Verify Deployment**: 
   After the pipeline completes, visit your DigitalOcean droplet’s IP address to verify the deployment:

Visit the deployed application at:

[(http://178.128.55.90/)](http://178.128.55.90/)

**Note**: If the droplet is not accessible, it may have been destroyed to reduce the DigitalOcean rental costs.


## Bonus: Deployment to DigitalOcean

The pipeline includes a step that connects to your DigitalOcean droplet via SSH, pulls the latest Docker image from Docker Hub, and runs it in a container. The application will be accessible on **port 80** of the droplet.

![image](https://github.com/user-attachments/assets/2835ecb6-21c4-4dd2-9a4e-3e895a65c4a2)





