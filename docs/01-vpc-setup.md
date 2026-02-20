# 01 – VPC Setup for ECS Fargate (Ritual Roast)

This document explains the complete **VPC networking setup** for the **Ritual Roast ECS Fargate project**. The architecture follows AWS best practices with proper network isolation, high availability, and secure traffic flow.

---

## 🧱 Architecture Overview

The VPC is divided into **three network layers**:

* **Public Subnets** – Application Load Balancer (ALB)
* **Private App Subnets** – ECS Fargate tasks
* **Private Data Subnets** – RDS MySQL database

### Traffic Flow

```
Internet
   |
[ Application Load Balancer ]   → Public Subnets
   |
[ ECS Fargate Tasks ]           → Private App Subnets
   |
[ RDS MySQL Database ]          → Private Data Subnets
```

---

## 1️⃣ Create VPC (VPC and More)

Navigate to:

**AWS Console → VPC → Create VPC → VPC and more**

### VPC Configuration

| Field     | Value            |
| --------- | ---------------- |
| Name      | Ritual-roast-vpc |
| IPv4 CIDR | 10.0.0.0/16      |
| Tenancy   | Default          |
| IPv6      | None             |

---

## 2️⃣ Availability Zones

| Setting            | Value                    |
| ------------------ | ------------------------ |
| Availability Zones | 2                        |
| Example AZs        | ap-south-1a, ap-south-1b |

Using multiple AZs ensures **high availability** and **fault tolerance**.

---

## 3️⃣ Subnet Configuration (Critical)

### Subnet Count

| Subnet Type        | Count |
| ------------------ | ----- |
| Public             | 2     |
| Private App (ECS)  | 2     |
| Private Data (RDS) | 2     |

---

### 🌐 Public Subnets (ALB)

| Name                | AZ          | CIDR        |
| ------------------- | ----------- | ----------- |
| rr-public-subnet-1a | ap-south-1a | 10.0.1.0/24 |
| rr-public-subnet-1b | ap-south-1b | 10.0.2.0/24 |

* Auto-assign public IP: **Enabled**
* Used for internet-facing ALB

---

### ⚙️ Private App Subnets (ECS Fargate)

| Name                     | AZ          | CIDR         |
| ------------------------ | ----------- | ------------ |
| rr-private-app-subnet-1a | ap-south-1a | 10.0.11.0/24 |
| rr-private-app-subnet-1b | ap-south-1b | 10.0.12.0/24 |

* Auto-assign public IP: **Disabled**
* Used by ECS Fargate tasks

---

### 🗄️ Private Data Subnets (RDS)

| Name                      | AZ          | CIDR         |
| ------------------------- | ----------- | ------------ |
| rr-private-data-subnet-1a | ap-south-1a | 10.0.21.0/24 |
| rr-private-data-subnet-1b | ap-south-1b | 10.0.22.0/24 |

* Auto-assign public IP: **Disabled**
* Used only for database layer

---

## 4️⃣ Internet Gateway

| Resource         | Name   |
| ---------------- | ------ |
| Internet Gateway | rr-igw |

Attach **rr-igw** to **Ritual-roast-vpc** to allow public subnet internet access.

---

## 5️⃣ NAT Gateways (Required)

To allow private subnets to access AWS services securely:

| NAT Gateway | Subnet              |
| ----------- | ------------------- |
| rr-natgw-1a | rr-public-subnet-1a |
| rr-natgw-1b | rr-public-subnet-1b |

* One NAT Gateway per AZ
* Required for ECS tasks to:

  * Pull images from ECR
  * Send logs to CloudWatch

---

## 6️⃣ Route Tables

### 🌐 Public Route Table

| Setting            | Value                                    |
| ------------------ | ---------------------------------------- |
| Name               | rr-public-rt                             |
| Route              | 0.0.0.0/0 → rr-igw                       |
| Associated Subnets | rr-public-subnet-1a, rr-public-subnet-1b |

---

### ⚙️ Private App Route Tables

| Route Table          | AZ | Route                   |
| -------------------- | -- | ----------------------- |
| rr-private-app-rt-1a | 1a | 0.0.0.0/0 → rr-natgw-1a |
| rr-private-app-rt-1b | 1b | 0.0.0.0/0 → rr-natgw-1b |

Associated with:

* rr-private-app-subnet-1a
* rr-private-app-subnet-1b

---

### 🗄️ Private Data Route Tables

| Route Table           | AZ | Route                   |
| --------------------- | -- | ----------------------- |
| rr-private-data-rt-1a | 1a | 0.0.0.0/0 → rr-natgw-1a |
| rr-private-data-rt-1b | 1b | 0.0.0.0/0 → rr-natgw-1b |

---

## 7️⃣ Security Groups

### 🔐 ALB Security Group – rr-alb-sg

| Rule     | Value                  |
| -------- | ---------------------- |
| Inbound  | HTTP 80 from 0.0.0.0/0 |
| Outbound | All traffic            |

---

### 🔐 App Security Group – rr-app-sg

| Rule     | Value                      |
| -------- | -------------------------- |
| Inbound  | All traffic from rr-alb-sg |
| Outbound | All traffic                |

---

### 🔐 Data Security Group – rr-data-sg

| Rule     | Value                               |
| -------- | ----------------------------------- |
| Inbound  | MySQL (3306) from rr-app-sg         |
| Inbound  | MySQL (3306) from rr-data-sg (self) |
| Outbound | All traffic                         |

**Security Group Chain**

```
ALB SG → App SG → Data SG
```

---

## 8️⃣ Final Verification Checklist ✅

* VPC created successfully
* 2 public subnets
* 2 private app subnets
* 2 private data subnets
* Internet Gateway attached
* NAT Gateway per AZ
* Correct route table associations
* Layered security groups

---

## 🎯 Outcome

This VPC provides a **production-grade network foundation** for ECS Fargate with:

* No public access to ECS or RDS
* Secure outbound internet via NAT
* High availability across AZs
* Clean separation of concerns

---


