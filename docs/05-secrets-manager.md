# 05 – AWS Secrets Manager Setup (Ritual Roast)

This document explains how to securely store and rotate **RDS MySQL credentials** using **AWS Secrets Manager** for the Ritual Roast ECS Fargate project. Secrets Manager eliminates hardcoded credentials and enables automatic password rotation using a managed Lambda function.

---

## 🎯 Objective

- Store RDS credentials securely
- Enable automatic secret rotation
- Allow secure Lambda-to-RDS access using security groups

---

## 1️⃣ Open AWS Secrets Manager

Navigate to:

**AWS Console → Secrets Manager → Store a new secret**

---

## 2️⃣ Secret Type

| Setting         | Value                               |
| --------------- | ----------------------------------- |
| Secret type     | Credentials for Amazon RDS database |
| Database engine | MySQL                               |

---

## 3️⃣ Enter Credentials

| Field    | Value                       |
| -------- | --------------------------- |
| Username | admin                       |
| Password | Same as RDS master password |

📌 These must match the credentials used while creating the RDS instance.

---

## 4️⃣ Select Database

| Field        | Value         |
| ------------ | ------------- |
| RDS database | ritualroastdb |

✔ AWS validates connectivity automatically
✔ Secret is linked directly with the RDS instance

Click **Next**.

---

## 5️⃣ Secret Name & Description

| Field       | Value                            |
| ----------- | -------------------------------- |
| Secret name | ritual-roast-db-secret           |
| Description | RDS credentials for Ritual Roast |

Click **Next**.

---

## 6️⃣ Configure Automatic Rotation

### Rotation Settings

| Field              | Value                       |
| ------------------ | --------------------------- |
| Automatic rotation | Enabled                     |
| Rotate every       | 7 days                      |
| Schedule type      | Schedule expression builder |

### Rotation Lambda

| Field                        | Value                        |
| ---------------------------- | ---------------------------- |
| Create new rotation function | Yes                          |
| Lambda function name         | ritual-roast-secret-rotation |
| Rotation strategy            | Single user                  |

📌 **Single-user rotation** updates the master password and is suitable for labs and free-tier setups.

Click **Next → Store secret**.

---

## 7️⃣ Resources Created Automatically by AWS

Secrets Manager automatically provisions:

- Lambda rotation function
- IAM role for Lambda
- Secret versions and rotation workflow

❌ No manual Lambda creation required.

---

## 8️⃣ Networking & Security Group Configuration (Critical)

### How Rotation Lambda Accesses RDS

- Rotation Lambda runs **inside the VPC**
- Lambda attaches an ENI
- Lambda uses **rr-data-sg**
- RDS also uses **rr-data-sg**

➡️ This requires a **self-referencing inbound rule**.

### Required Security Group Rule

| Type         | Port | Source     |
| ------------ | ---- | ---------- |
| MySQL/Aurora | 3306 | rr-data-sg |

✔ Allows Lambda → RDS communication
✔ Does not expose database publicly

---

## 🔐 Final Security Group Chain

| From       | To         | Allowed |
| ---------- | ---------- | ------- |
| rr-app-sg  | rr-data-sg | ✅      |
| rr-data-sg | rr-data-sg | ✅      |
| Internet   | rr-data-sg | ❌      |

---

## 9️⃣ Verify Rotation Status

Navigate to:

**Secrets Manager → ritual-roast-db-secret**

Verify:

- Rotation status: **Enabled**
- Last rotation: Successful
- Next scheduled rotation date

---

## 🧠 Interview-Ready Explanation

**Q: How does Secrets Manager rotate RDS credentials securely?**

✅ Secrets Manager uses a Lambda function deployed inside the VPC to rotate database passwords automatically. Access to RDS is controlled using security groups with self-referencing rules, ensuring no public exposure.

---

## ✅ Outcome

- Database credentials removed from application code
- Automatic password rotation enabled
- Lambda created and managed by AWS
- Secure VPC-based access to RDS
- Production-grade secret management

---
