# 04 – RDS MySQL Setup (Ritual Roast)

This document explains how to create a **private Amazon RDS MySQL database** for the Ritual Roast application. The database is deployed in **private data subnets**, accessible only from ECS Fargate tasks, following AWS security best practices.

---

## 🧱 Architecture Context

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

## 🎯 Objective

* Create a DB subnet group using private data subnets
* Launch a MySQL RDS instance without public access
* Restrict access using security groups

---

## 1️⃣ Create DB Subnet Group

Navigate to:

**AWS Console → RDS → Subnet groups → Create DB subnet group**

### Subnet Group Configuration

| Field       | Value                             |
| ----------- | --------------------------------- |
| Name        | ritual-roast-db-subnet-group      |
| Description | Subnet group for Ritual Roast RDS |
| VPC         | Ritual-roast-vpc                  |

### Subnets Selection (Critical)

Select **only private data subnets**:

* rr-private-data-subnet-1a
* rr-private-data-subnet-1b

❌ Do not select public or app subnets.

Click **Create**.

✅ DB subnet group created successfully.

---

## 2️⃣ Create RDS MySQL Database

Navigate to:

**AWS Console → RDS → Databases → Create database**

---

### Database Creation Method

| Setting         | Value           |
| --------------- | --------------- |
| Creation method | Standard create |

---

### Engine Options

| Field   | Value         |
| ------- | ------------- |
| Engine  | MySQL         |
| Version | Default (8.x) |

---

### Templates

| Field    | Value     |
| -------- | --------- |
| Template | Free tier |

---

### DB Settings

| Field                  | Value           |
| ---------------------- | --------------- |
| DB instance identifier | ritualroastdb   |
| Master username        | admin           |
| Master password        | AnyGoodPassword |

📌 Credentials will be moved to **AWS Secrets Manager** later.

---

### Instance Configuration

| Field             | Value           |
| ----------------- | --------------- |
| DB instance class | db.t3.micro     |
| Storage           | Default (20 GB) |

---

### Connectivity (Very Important)

| Field              | Value                        |
| ------------------ | ---------------------------- |
| Compute resource   | Don’t connect to EC2         |
| VPC                | Ritual-roast-vpc             |
| Subnet group       | ritual-roast-db-subnet-group |
| Public access      | No                           |
| VPC security group | rr-data-sg                   |

✔ Database is private
✔ Accessible only from ECS app layer

---

### Database Options

| Field                 | Value         |
| --------------------- | ------------- |
| Initial database name | ritualroastdb |

---

### Monitoring & Backup

Keep default settings (safe for Free Tier).

Click **Create database**.

⏳ Creation time: ~5–10 minutes

---

## 3️⃣ Verify Database

Once database status is **Available**, note the following:

* RDS endpoint
* Port: **3306**

Example:

```
ritualroastdb.xxxxxx.ap-south-1.rds.amazonaws.com
```

---

## 🔐 Security Flow (Interview-Ready)

| Source      | Destination | Allowed |
| ----------- | ----------- | ------- |
| Internet    | RDS         | ❌       |
| ALB         | RDS         | ❌       |
| ECS Fargate | RDS         | ✅       |
| EC2         | RDS         | ❌       |

This is enforced using **rr-data-sg**, which allows traffic **only from rr-app-sg**.

---

## 🧠 Interview-Ready Explanation

**Q: Why is RDS deployed in private subnets?**

✅ Deploying RDS in private subnets prevents direct internet access, reduces attack surface, and ensures that only application services inside the VPC can communicate with the database.

---

## ✅ Outcome

* MySQL RDS deployed securely
* No public database access
* Multi-AZ ready subnet design
* Secure communication with ECS Fargate
* Production-grade database setup

---

