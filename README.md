# Brain Tasks App – DevOps Deployment Project

## Project Overview

This project demonstrates the deployment of a production-ready web application using modern DevOps practices and AWS cloud services.

The application source repository contains only the production build (`dist` folder). The application was containerized using Docker, stored in Amazon Elastic Container Registry (ECR), deployed on Amazon Elastic Kubernetes Service (EKS), and automated through AWS CodeBuild and CodePipeline.

## Architecture Diagram

GitHub Repository
↓
AWS CodePipeline
↓
AWS CodeBuild
↓
Docker Image Build
↓
Amazon ECR
↓
Amazon EKS
↓
Kubernetes LoadBalancer
↓
Application Access

## Application Information

Repository:
https://github.com/Vennilavanguvi/Brain-Tasks-App

Application Type:
Static web application (production build in dist folder)

Deployment Port:
3000 (local Docker testing)

Container Port:
80 (Nginx)

## Docker Setup

### Dockerfile

```dockerfile
FROM nginx:alpine

COPY dist/ /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Build Docker Image

```bash
docker build -t brain-task-app .
```

### Run Docker Container

```bash
docker run -d -p 3000:80 brain-task-app
```

### Verify

```bash
docker ps
```

Application URL:

http://localhost:3000

## Amazon ECR Setup

### Create Repository

```bash
aws ecr create-repository \
--repository-name brain-task-app \
--region ap-south-1
```

### Login to ECR

```bash
aws ecr get-login-password \
--region ap-south-1 \
| docker login \
--username AWS \
--password-stdin 155326049312.dkr.ecr.ap-south-1.amazonaws.com
```

### Push Docker Image

```bash
docker tag brain-task-app:latest \
155326049312.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest

docker push \
155326049312.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest
```

## Amazon EKS Setup

### Create EKS Cluster

```bash
eksctl create cluster \
--name brain-task-cluster \
--region ap-south-1 \
--nodegroup-name workers \
--node-type t3.small \
--nodes 1
```

### Verify Cluster

```bash
kubectl get nodes
```

## Kubernetes Deployment

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: brain-task-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: brain-task-app

  template:
    metadata:
      labels:
        app: brain-task-app

    spec:
      containers:
      - name: brain-task-app
        image: 155326049312.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest

        ports:
        - containerPort: 80
```

### service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: brain-task-service

spec:
  selector:
    app: brain-task-app

  ports:
  - port: 80
    targetPort: 80

  type: LoadBalancer
```

### Deploy Application

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Verify Deployment

```bash
kubectl get pods
kubectl get svc
```

## AWS CodeBuild Setup

CodeBuild was configured to:

1. Pull source code from GitHub
2. Build Docker image
3. Push image to Amazon ECR
4. Deploy Kubernetes manifests to Amazon EKS

### buildspec.yml

```yaml
version: 0.2

phases:

  pre_build:
    commands:
      - aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com

  build:
    commands:
      - docker build -t brain-task-app .
      - docker tag brain-task-app:latest $AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest

  post_build:
    commands:
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest
      - kubectl apply -f deployment.yaml
      - kubectl apply -f service.yaml
```

## AWS CodePipeline Setup

Pipeline Stages:

### Source Stage

Provider: GitHub

Repository: brain-task-app

Branch: main

### Build Stage

Provider: AWS CodeBuild

Project: brain-task-build

### Deploy Stage

Deployment executed through CodeBuild using kubectl commands.

### CI/CD Flow

GitHub Commit
↓
CodePipeline Trigger
↓
CodeBuild Execution
↓
Docker Build
↓
Push to ECR
↓
Deploy to EKS
↓
Application Updated

## CloudWatch Monitoring

CloudWatch was used to monitor:

* CodeBuild logs
* Build status
* Deployment execution logs
* Pipeline execution events

Log Group Example:

/aws/codebuild/brain-task-build

Monitoring Steps:

1. Open CloudWatch Console
2. Navigate to Log Groups
3. Open CodeBuild log group
4. Review build and deployment logs

## Application Access

LoadBalancer URL:

http://a5d5e659a1c3640db8f15107764637bc-607505989.ap-south-1.elb.amazonaws.com

## Conclusion

The application was successfully containerized using Docker, stored in Amazon ECR, deployed on Amazon EKS, and automated using AWS CodeBuild and CodePipeline. CloudWatch was used for monitoring and logging, providing a complete CI/CD deployment workflow on AWS.
