# CI/CD Pipeline with Docker and GitHub Actions

This repository demonstrates a basic CI/CD pipeline setup using GitHub Actions, Docker, and DigitalOcean. The pipeline builds a Docker image from the application, pushes it to Docker Hub, and then deploys it to a DigitalOcean droplet via SSH.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [GitHub Secrets](#github-secrets)
- [DigitalOcean Droplet Setup](#digitalocean-droplet-setup)
- [CI/CD Workflow](#cicd-workflow)
- [Testing the Workflow](#testing-the-workflow)
- [Troubleshooting](#troubleshooting)

## Prerequisites

To replicate this setup, you will need:

- A GitHub account and a repository.
- A Docker Hub account to store the Docker images.
- A DigitalOcean account to create and manage a droplet.
- An SSH key pair to connect to the droplet securely.

## Setup

1. **Create SSH Key Pair**:
   Generate an SSH key pair to use for authentication with the DigitalOcean droplet.
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
    The private key (id_rsa) will be used in GitHub Secrets.
    The public key (id_rsa.pub) will be added to the DigitalOcean droplet.


2. **Configure Docker**:
   Ensure you have a Dockerfile in your application repository to build the Docker image.



3. **Configure GitHub Actions Workflow**: A GitHub Actions workflow file (.github/workflows/main.yml) is provided in this repository to handle the CI/CD pipeline. This file builds the Docker image, pushes it to Docker Hub, and deploys it to the DigitalOcean droplet.


