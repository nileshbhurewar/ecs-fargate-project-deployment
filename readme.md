# AWS ECS Fargate End-to-End Deployment Project

## Project Overview

This repository documents the manual deployment of a containerized application on AWS using Amazon ECS with the Fargate launch type. The project demonstrates how to build a Docker image, push it to Amazon ECR, provision required AWS infrastructure, and deploy the application behind an Application Load Balancer.

This project focuses on core AWS container services and networking concepts without using CI/CD automation.

The main objective is to understand how ECS Fargate works internally, how containers are managed in AWS, and how different AWS services integrate together in a production-style environment.

---

## Architecture Summary
![Architecture](https://github.com/nileshbhurewar/ecs-fargate-project-deployment/blob/main/architecture-diagram/diagram.png)


The deployment architecture includes the following components:

* Docker for containerization
* Amazon ECR for storing container images
* Amazon ECS (Fargate) for running containers
* Application Load Balancer (ALB) for traffic routing
* Amazon RDS (MySQL) for database
* IAM roles for secure service permissions
* VPC, Subnets, Security Groups for networking

Application Flow:

1. Application is containerized using Docker.
2. Docker image is pushed to Amazon ECR.
3. ECS Task Definition references the ECR image.
4. ECS Service runs the task using Fargate.
5. ALB forwards incoming traffic to running ECS tasks.
6. Application connects to Amazon RDS database.

---

## Prerequisites

Before starting the deployment, ensure the following tools and accounts are available:

* AWS Account
* IAM user with programmatic access
* AWS CLI installed and configured
* Docker installed
* Git installed

Configure AWS CLI using:

```
aws configure
```

Provide:

* AWS Access Key ID
* AWS Secret Access Key
* Default region (for example: ap-south-1)
* Output format (json)

---

## Step 1: Dockerize the Application

Build Docker image:

```
docker build -t my-app .
```

Verify image creation:

```
docker images
```

---

## Step 2: Create Amazon ECR Repository

Create repository:

```
aws ecr create-repository --repository-name my-app
```

Authenticate Docker with ECR:

```
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-south-1.amazonaws.com
```

Tag Docker image:

```
docker tag my-app:latest <account-id>.dkr.ecr.ap-south-1.amazonaws.com/my-app:latest
```

Push image to ECR:

```
docker push <account-id>.dkr.ecr.ap-south-1.amazonaws.com/my-app:latest
```

---

## Step 3: Create Amazon RDS (MySQL)

* Engine: MySQL
* Select appropriate instance size (Free tier if applicable)
* Configure database name, username, and password
* Configure security group to allow access from ECS tasks

Note the RDS endpoint and update application configuration accordingly.

---

## Step 4: Create ECS Cluster

1. Open ECS Console
2. Select Create Cluster
3. Choose Networking only (Fargate)
4. Provide cluster name
5. Create cluster

---

## Step 5: Create Task Definition

* Launch Type: Fargate
* Define CPU and Memory configuration
* Add container definition
* Provide ECR image URI
* Set container port
* Configure environment variables (DB_HOST, DB_USER, DB_PASSWORD, DB_NAME)
* Assign execution role with ECR and CloudWatch permissions

Register the task definition.

---

## Step 6: Create ECS Service

* Select cluster
* Choose task definition
* Launch type: Fargate
* Configure desired task count
* Configure VPC, subnets, and security groups
* Attach Application Load Balancer
* Select target group

Create the service and wait for tasks to reach running state.

---

## Step 7: Configure Application Load Balancer

* Create ALB
* Configure HTTP listener (Port 80)
* Create target group (IP target type for Fargate)
* Attach target group to ECS service

After deployment completes, access the application using the ALB DNS name.

---

## Networking Configuration

The project uses:

* Custom or default VPC
* Public subnets for ALB
* Private or public subnets for ECS tasks (based on configuration)
* Security groups to allow:

  * HTTP traffic to ALB
  * ALB to ECS tasks
  * ECS tasks to RDS

Proper security group configuration is required for successful communication between services.

---

## IAM Roles Used

ECS Task Execution Role must include permissions for:

* AmazonEC2ContainerRegistryReadOnly
* CloudWatchLogsFullAccess

Ensure least privilege access is followed in production environments.

---

## Verification Steps

1. Confirm ECS service is stable.
2. Ensure tasks are running.
3. Check target group health status.
4. Access application using ALB DNS URL.
5. Verify database connectivity.

---

## Cleanup Steps

To avoid unnecessary AWS charges, delete resources in the following order:

1. ECS Service
2. ECS Cluster
3. Load Balancer
4. Target Groups
5. RDS Instance
6. ECR Repository
7. Security Groups (if created separately)

---

## Learning Outcomes

Through this project, the following concepts are demonstrated:

* Containerization using Docker
* Image management with Amazon ECR
* Serverless container deployment with ECS Fargate
* Load balancing using ALB
* Secure networking using VPC and security groups
* Database integration with Amazon RDS
* IAM role-based access control

---

## Project Status

Manual deployment completed successfully using AWS Console and AWS CLI.

---

## Author

Nilesh Bhurewar

This repository is created for learning and demonstrating AWS container deployment concepts.
