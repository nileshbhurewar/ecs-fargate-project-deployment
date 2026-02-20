# 06 – Application Load Balancer & Target Group (Ritual Roast)

This document explains how to configure the **Application Load Balancer (ALB)** and **Target Group** for the Ritual Roast ECS Fargate application.

The ALB distributes incoming HTTP traffic from the internet to ECS tasks running inside private subnets.

---

## 🧱 Architecture Context

```
Internet
   |
[ ritual-roast-alb ]       → Public Subnets
          |
[ Target Group (IP mode) ]
          |
[ ECS Fargate Tasks ]      → Private App Subnets
```

---

## 🎯 Objective

* Create a Target Group using IP target type
* Configure health checks
* Create an Internet-facing Application Load Balancer
* Connect ALB to ECS service

---

# 1️⃣ Create Target Group

Navigate to:

**AWS Console → EC2 → Target Groups → Create target group**

---

## Target Group Configuration

| Field             | Value            |
| ----------------- | ---------------- |
| Target type       | IP addresses     |
| Target group name | ritual-roast-tg  |
| Protocol          | HTTP             |
| Port              | 80               |
| VPC               | Ritual-roast-vpc |

📌 **Important:**
ECS Fargate tasks do not run on EC2 instances. Each task gets its own ENI and IP address, so **IP mode is mandatory**.

---

## Health Check Configuration

| Field               | Value        |
| ------------------- | ------------ |
| Protocol            | HTTP         |
| Health check path   | /health.html |
| Port                | traffic port |
| Healthy threshold   | 2            |
| Unhealthy threshold | 3            |
| Timeout             | 5 seconds    |
| Interval            | 30 seconds   |
| Success codes       | 200          |

✔ Ensures only healthy ECS tasks receive traffic.

---

## Register Targets

Do NOT manually register targets.

You will see:

* 0 targets registered (This is correct)

📌 ECS service will automatically register tasks.

Click **Create target group**.

---

# 2️⃣ Create Application Load Balancer

Navigate to:

**AWS Console → EC2 → Load Balancers → Create Load Balancer**

---

## Load Balancer Type

| Field | Value                     |
| ----- | ------------------------- |
| Type  | Application Load Balancer |

---

## Basic Configuration

| Field           | Value            |
| --------------- | ---------------- |
| Name            | ritual-roast-alb |
| Scheme          | Internet-facing  |
| IP address type | IPv4             |

---

## Network Mapping

| Field   | Value                                    |
| ------- | ---------------------------------------- |
| VPC     | Ritual-roast-vpc                         |
| Subnets | rr-public-subnet-1a, rr-public-subnet-1b |

✔ ALB must be deployed in **public subnets**
✔ Ensures high availability across AZs

---

## Security Group

| Field          | Value     |
| -------------- | --------- |
| Security group | rr-alb-sg |

Security Group Rule:

* Inbound: HTTP (80) from 0.0.0.0/0
* Outbound: All traffic

✔ Only ALB is exposed to the internet

---

## Listeners & Routing

| Field      | Value           |
| ---------- | --------------- |
| Listener   | HTTP : 80       |
| Forward to | ritual-roast-tg |

ALB will forward incoming requests to the Target Group.

Click **Create Load Balancer**.

⏳ Provisioning time: 1–2 minutes

---

# 3️⃣ Verify ALB

After creation:

* State = Active
* Copy the ALB DNS name

Example:

```
ritual-roast-alb-xxxx.ap-south-1.elb.amazonaws.com
```

⚠ If you open the DNS now, you may see:

* 503 Service Unavailable

✅ This is expected because ECS tasks are not deployed yet.

---

## 🔐 Traffic Flow & Security Model

| Source    | Destination | Allowed                    |
| --------- | ----------- | -------------------------- |
| Internet  | ALB         | ✅                          |
| ALB       | ECS Tasks   | (After service creation) ✅ |
| ECS Tasks | RDS         | ✅                          |
| Internet  | ECS         | ❌                          |
| Internet  | RDS         | ❌                          |

This layered approach ensures proper isolation.

---

## 🧠 Interview-Ready Explanation

**Q: Why use IP-based Target Groups for ECS Fargate?**

✅ ECS Fargate tasks do not run on EC2 instances. Each task receives its own Elastic Network Interface (ENI) and private IP address. Therefore, the Target Group must use IP mode instead of instance mode.

---

## ✅ Outcome

* Application Load Balancer created
* Target Group configured correctly
* Health checks aligned with application
* Secure internet entry point established
* Ready for ECS service integration

---


