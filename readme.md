# AWS ECS + Fargate End-to-End Project Deployment

This repository documents a complete end-to-end deployment of a containerized application on AWS using ECS Fargate.

The goal of this project is to demonstrate how production-grade container workloads are deployed securely and scalably on AWS.

---

## 🧱 Architecture Overview

![Architecture](https://github.com/nileshbhurewar/ecs-fargate-project-deployment/blob/main/architecture-diagram/diagram.png)

---

## 🔧 Services Used

- Amazon ECS (Fargate)
- Amazon ECR
- Application Load Balancer
- Amazon RDS (MySQL)
- AWS Secrets Manager (with rotation)
- AWS IAM
- Amazon VPC (3-tier architecture)
- Amazon CloudWatch

---

## 🚀 Project Flow

1. Designed a custom VPC with public and private subnets across two AZs
2. Built and pushed Docker image to Amazon ECR
3. Created RDS MySQL database in private subnets
4. Secured DB credentials using AWS Secrets Manager
5. Created IAM roles for ECS tasks
6. Configured ALB and target groups
7. Deployed ECS Fargate service with auto scaling
8. Verified application through ALB endpoint
9. Cleaned up all resources to avoid costs


