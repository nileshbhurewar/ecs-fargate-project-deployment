# 08 – ECS Task Definition & Service Deployment (Ritual Roast)

This document explains how to create the **ECS Task Definition** and deploy the **ECS Service** using AWS Fargate for the Ritual Roast application.

This is the step where the containerized application becomes live.

---

## 🧠 Important IAM Clarification

ECS uses two roles:

| Role           | Used For                                                  |
| -------------- | --------------------------------------------------------- |
| Task Role      | Application access to AWS services (Secrets Manager)      |
| Execution Role | ECS infrastructure operations (ECR pull, CloudWatch logs) |

For this project:

- **Task Role** → `ritual-roast-ecs-task-role`
- **Execution Role** → `ecsTaskExecutionRole` (AWS managed)

---

# 1️⃣ Create ECS Task Definition

Navigate to:

**AWS Console → ECS → Task Definitions → Create new task definition**

---

## Task Definition Configuration

| Field            | Value                        |
| ---------------- | ---------------------------- |
| Family name      | ritual-roast-task-definition |
| Launch type      | AWS Fargate                  |
| Operating system | Linux                        |
| CPU              | 0.5 vCPU                     |
| Memory           | 1 GB                         |

---

## IAM Roles

| Field               | Value                      |
| ------------------- | -------------------------- |
| Task role           | ritual-roast-ecs-task-role |
| Task execution role | ecsTaskExecutionRole       |

---

# 2️⃣ Container Configuration

## Container Details

| Field          | Value                                                           |
| -------------- | --------------------------------------------------------------- |
| Container name | ritualroast                                                     |
| Image URI      | <account-id>.dkr.ecr.<region>.amazonaws.com/ritual-roast:latest |
| Essential      | Yes                                                             |
| Port mapping   | 80 (HTTP)                                                       |

---

## Environment Variables

| Key         | Value                  |
| ----------- | ---------------------- |
| DB_SERVER   | RDS endpoint           |
| DB_DATABASE | ritualroastdb          |
| SECRET_NAME | ritual-roast-db-secret |
| AWS_REGION  | ap-south-1             |

📌 Region must match where the secret exists.

---

## Logging Configuration (Important)

| Field         | Value             |
| ------------- | ----------------- |
| Log driver    | awslogs           |
| Log group     | /ecs/ritual-roast |
| Stream prefix | ecs               |

✔ Enables centralized logging in CloudWatch
✔ Required for debugging and monitoring

Click **Create Task Definition**.

---

# 3️⃣ Create ECS Service

Navigate to:

**ECS → Clusters → ritual-roast-ecs-cluster → Services → Create**

---

## Service Configuration

| Field            | Value                          |
| ---------------- | ------------------------------ |
| Launch type      | Fargate                        |
| Application type | Service                        |
| Task definition  | ritual-roast-task-definition:1 |
| Service name     | ritual-roast-service           |
| Desired tasks    | 1                              |

---

## Networking Configuration

| Field          | Value                                              |
| -------------- | -------------------------------------------------- |
| VPC            | Ritual-roast-vpc                                   |
| Subnets        | rr-private-app-subnet-1a, rr-private-app-subnet-1b |
| Security group | rr-app-sg                                          |
| Public IP      | Disabled                                           |

✔ Tasks remain private
✔ Only accessible via ALB

---

## Load Balancer Configuration

| Field              | Value                     |
| ------------------ | ------------------------- |
| Load balancer type | Application Load Balancer |
| Load balancer      | ritual-roast-alb          |
| Listener           | HTTP : 80                 |
| Target group       | ritual-roast-tg           |

ECS automatically registers tasks to the target group.

---

# 4️⃣ Configure Service Auto Scaling

Enable **Service Auto Scaling**.

## Scaling Configuration

| Field                 | Value                           |
| --------------------- | ------------------------------- |
| Minimum tasks         | 1                               |
| Maximum tasks         | 4                               |
| Scaling type          | Target tracking                 |
| Metric                | ECSServiceAverageCPUUtilization |
| Target value          | 70%                             |
| Scale in/out cooldown | 300 seconds                     |

✔ Scales out when CPU > 70%
✔ Scales in when CPU drops

---

# 5️⃣ Verify Deployment

After creation:

- Service status = Active
- Tasks = Running
- Target group = Healthy targets
- Logs visible in CloudWatch

---

# 6️⃣ Access Application

Copy ALB DNS name:

```
http://ritual-roast-alb-xxxx.ap-south-1.elb.amazonaws.com
```

🎉 Ritual Roast application is now LIVE on ECS Fargate.

---

## 🧠 Interview-Ready Summary

I deployed a containerized web application on AWS ECS Fargate using a custom VPC, Application Load Balancer, Secrets Manager for database credentials, RDS in private subnets, and implemented auto scaling based on CPU utilization.

---

## ✅ Outcome

- ECS Task Definition created
- Secure IAM integration implemented
- ECS Service deployed
- Auto scaling enabled
- Application publicly accessible via ALB
- Production-grade architecture completed

---
