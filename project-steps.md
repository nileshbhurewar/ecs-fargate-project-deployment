📦 AWS ECS + Fargate End-to-End Project Deployment

This repository documents a complete end-to-end deployment of a containerized application on AWS ECS using Fargate.

The project demonstrates how to design and deploy a production-grade cloud architecture using AWS managed services with security, scalability, and best practices.

🏗️ Architecture Overview

High-level architecture:

Internet
   |
Application Load Balancer (Public Subnets)
   |
ECS Fargate Service (Private App Subnets)
   |
RDS MySQL (Private Data Subnets)

🔧 AWS Services Used

Amazon VPC

Amazon ECS (Fargate)

Amazon ECR

Application Load Balancer

Amazon RDS (MySQL)

AWS Secrets Manager (with rotation)

AWS IAM

AWS CloudWatch

AWS Systems Manager (Session Manager)

🚀 Project Implementation – Step by Step
1️⃣ VPC Setup (3-Tier Architecture)
VPC Configuration

VPC Name: Ritual-roast-vpc

CIDR: 10.0.0.0/16

AZs: 2 (example: ap-south-1a, ap-south-1b)

Subnets

Public Subnets

rr-public-subnet-1a → 10.0.1.0/24

rr-public-subnet-1b → 10.0.2.0/24

Private App Subnets (ECS)

rr-private-app-subnet-1a → 10.0.11.0/24

rr-private-app-subnet-1b → 10.0.12.0/24

Private Data Subnets (RDS)

rr-private-data-subnet-1a → 10.0.21.0/24

rr-private-data-subnet-1b → 10.0.22.0/24

Networking Components

Internet Gateway: rr-igw

NAT Gateway: 1 per AZ

Route tables for public, app, and data layers

Security Groups

rr-alb-sg → Internet → ALB (HTTP 80)

rr-app-sg → ALB → ECS

rr-data-sg → ECS → RDS + self-reference (for Secrets rotation Lambda)

2️⃣ Create Amazon ECR Repository

Repository Name: ritual-roast

Visibility: Private

Image scanning: Enabled

Encryption: AES-256

This repository stores the Docker image used by ECS Fargate.

3️⃣ Build & Push Docker Image
EC2 Build Server

Ubuntu 22.04 (private subnet)

No SSH access (Session Manager only)

IAM Role permissions:

AmazonSSMManagedInstanceCore

EC2InstanceProfileForImageBuilderECRContainerBuilds

Steps
docker build -t ritual-roast .
docker tag ritual-roast <ECR-URI>
docker push <ECR-URI>

4️⃣ Create RDS MySQL Database
Database Configuration

Engine: MySQL

Instance: db.t3.micro

DB Name: ritualroastdb

Public access: ❌ No

Subnet group: ritual-roast-db-subnet-group

Security Group: rr-data-sg

Database runs fully inside private data subnets.

5️⃣ Configure AWS Secrets Manager

Secret Name: ritual-roast-db-secret

Stores RDS username & password

Automatic rotation enabled (every 7 days)

Rotation Lambda created automatically by AWS

Security group allows self-reference so rotation Lambda can access RDS.

6️⃣ Create Target Group & ALB
Target Group

Target type: IP

Protocol: HTTP

Health check path: /health.html

Application Load Balancer

Internet-facing

Deployed in public subnets

Listener: HTTP : 80

Forwards traffic to ECS target group

7️⃣ IAM Roles & ECS Cluster
IAM Roles

Task Role: Access Secrets Manager

Execution Role: Pull images & write logs

ECS Cluster

Name: ritual-roast-ecs-cluster

Launch type: AWS Fargate

8️⃣ ECS Task Definition & Service
Task Definition

CPU: 0.5 vCPU

Memory: 1 GB

Container port: 80

Environment variables:

DB_SERVER

DB_DATABASE

SECRET_NAME

AWS_REGION

ECS Service

Desired tasks: 1

Auto scaling enabled (1–4 tasks)

CPU target tracking

Integrated with ALB

🌐 Application Access

Once deployed, the application is accessible via:

http://<ALB-DNS-NAME>

🧹 Cleanup (Cost Control)

All resources are deleted in dependency order:

ECS Service & Cluster

Load Balancer & Target Group

RDS & Secrets

ECR Repository

EC2 Build Server

IAM Roles & Policies

VPC & Networking

This avoids orphaned billable resources.

🎯 Key Learnings

Designing secure multi-AZ VPC architectures

Running containers on ECS Fargate

Secure secrets handling using AWS Secrets Manager

Auto scaling and load balancing

IAM role separation (task vs execution role)

Cost-aware cloud resource cleanup

📌 Notes

No credentials or secrets are stored in this repository

Project is created for learning and demonstration purposes

Architecture follows AWS best practices

If you want, next I can:

✅ Split this into multiple markdown files
✅ Create a GitHub repo description + tags
✅ Review before you publish
✅ Write the LinkedIn post that links this repo

Just tell me 👍