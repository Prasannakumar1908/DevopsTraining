# DevopsTraining
Working with AWS CLI, eksctl, kubectl


# AWS CLI & Docker Image Deployment to EKS using WSL

## Table of Contents
1. [Introduction](#introduction)
2. [Pre-requisites](#pre-requisites)
3. [Installation Steps](#installation-steps)
   - [1. Install AWS CLI](#1-install-aws-cli)
   - [2. Configure AWS CLI](#2-configure-aws-cli)
   - [3. Install eksctl and kubectl](#3-install-eksctl-and-kubectl)
   - [4. Create an EKS Cluster](#4-create-an-eks-cluster)
   - [5. Build and Push Docker Image to AWS ECR](#5-build-and-push-docker-image-to-aws-ecr)
   - [6. Deploy Application to EKS](#6-deploy-application-to-eks)
4. [Troubleshooting](#troubleshooting)

## Introduction

This guide provides step-by-step instructions to set up the AWS CLI, deploy a Docker image to an EKS (Elastic Kubernetes Service) cluster, and manage your infrastructure using **WSL** (Windows Subsystem for Linux). We will install necessary tools such as **AWS CLI**, **Docker**, **eksctl**, and **kubectl**, followed by pushing a Docker image to AWS ECR and deploying it to an EKS cluster.

# Deploy Docker Image to AWS EKS Using WSL

This guide provides step-by-step instructions to deploy a Docker image to AWS Elastic Kubernetes Service (EKS) using Windows Subsystem for Linux (WSL). 

## Prerequisites

Ensure you have the following:
- Windows with WSL with Ubuntu 22.04 installed (Ubuntu or any other supported distribution)
   -To install Ubuntu 22.04 in your pc, run the following command
  ```bash
  wsl --install -d Ubuntu-22.04
  ```
  After successfully installing Ubuntu, you are prompted to enter username and password. Enter your username and password. Username should be in lowercase. If you want your username in Uppercase run it by force command.
  Now run the following commands
  ```bash
  sudo apt update
  ```
  You will be prompted to enter the password you set in the above step
  After this step is completed, run the following command
  ```bash
  sudo apt upgrade -y
  ```
  After you are done with these steps, now check the versions of apt and sudo by using the following commands.
  ```bash
  apt --version
  ```
  ```bash
  sudo --version
  ```
  
  
- AWS Account with sufficient permissions to create and manage EKS clusters
- AWS CLI installed on WSL
- Docker installed on WSL
- eksctl and kubectl installed on WSL

---

## Steps

### 1. Install AWS CLI on WSL

1. Update the package manager:
   ```bash
   sudo apt update
   ```
2. Install AWS CLI:
   ```bash
   sudo apt install awscli -y
   ```
3. Verify the installation:
   ```bash
   aws --version
   ```

### 2. Configure AWS CLI

1. Configure AWS CLI using the command:
   ```bash
   aws configure
   ```
2. Input your AWS Access Key ID, Secret Access Key, Default region, and Output format.

### 3. Install `eksctl` and `kubectl`

#### Install `eksctl`:

1. Download and install `eksctl`:
   ```bash
   curl -sSL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   eksctl version
   ```
Run the above commands separately. Command in each line should be run separately otherwise it won't work. Error occurs if the first command is wrong. So,ensure you are using the correct environment.

#### Install `kubectl`:

1. Download and install `kubectl`:
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   kubectl version --client
   ```
Run the above commands separately. Command in each line should be run separately otherwise it won't work. 

### 4. Create an EKS Cluster

1. Use `eksctl` to create a new EKS cluster:
   ```bash
   eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes
   ```
2. Verify the nodes in the cluster:
   ```bash
   kubectl get nodes
   ```

### 5. Build and Push Docker Image to AWS ECR

#### Build Docker Image:

1. Build your Docker image:
   ```bash
   docker build -t my-app-image .
   ```

#### Push Docker Image to AWS ECR:

1. Authenticate Docker to AWS ECR:
   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <your-aws-account-id>.dkr.ecr.us-east-1.amazonaws.com
   ```
2. Tag and push the Docker image:
   ```bash
   docker tag my-app-image:latest <your-aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
   docker push <your-aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
   ```

### 6. Deploy Application to EKS

1. Create a `deployment.yaml` file:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-app
           image: <your-aws-account-id>.dkr.ecr.us-east-1.amazonaws.com/my-repo:latest
           ports:
           - containerPort: 80
   ```

2. Apply the deployment to the cluster:
   ```bash
   kubectl apply -f deployment.yaml
   ```

### 7. Expose the Application

1. Create a `service.yaml` file to expose the application:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: my-app-service
   spec:
     type: LoadBalancer
     selector:
       app: my-app
     ports:
     - protocol: TCP
       port: 80
       targetPort: 80
   ```

2. Apply the service:
   ```bash
   kubectl apply -f service.yaml
   ```

3. Get the external IP of the load balancer:
   ```bash
   kubectl get services
   ```

### 8. Access the Application

1. Once you retrieve the external IP from the load balancer, you can open the URL in your browser to access the deployed application.

---

### Additional Resources

- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
- [eksctl Documentation](https://eksctl.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)

---

This guide covers the setup, configuration, and deployment process of a Dockerized application on AWS EKS using WSL.
