# 02 – Amazon ECR Setup (Ritual Roast)

This document explains how to create and configure an **Amazon Elastic Container Registry (ECR)** repository for the **Ritual Roast ECS Fargate project**. The ECR repository is used to securely store Docker images that will be pulled by ECS tasks.

---

## 🎯 Objective

* Create a **private ECR repository**
* Enable image scanning for security
* Prepare the repository for ECS integration

---

## 1️⃣ Open Amazon ECR

Navigate to:

**AWS Console → Services → Amazon Elastic Container Registry (ECR)**

---

## 2️⃣ Create ECR Repository

Click **Create repository** and configure the following settings.

### Repository Settings

| Field               | Value                 |
| ------------------- | --------------------- |
| Visibility settings | Private               |
| Repository name     | ritual-roast          |
| Tag immutability    | Disabled              |
| Scan on push        | Enabled (Recommended) |
| Encryption          | AES-256 (Default)     |

Click **Create repository**.

✅ The repository is now created successfully.

---

## 3️⃣ Verify Repository

After creation, verify the following details:

| Field           | Example                                                  |
| --------------- | -------------------------------------------------------- |
| Repository name | ritual-roast                                             |
| Repository URI  | <account-id>.dkr.ecr.<region>.amazonaws.com/ritual-roast |

📌 **Important**: Copy and save the **Repository URI**. It will be required while:

* Building Docker images
* Creating ECS Task Definitions

---

## 4️⃣ Why Use a Private ECR Repository?

Using a **private ECR repository** ensures:

* Only authorized IAM roles can push or pull images
* Secure storage of production Docker images
* Seamless integration with ECS Fargate
* No need to manage external container registries

---

## 🧠 Interview-Ready Explanation

**Q: Why did you choose Amazon ECR instead of Docker Hub?**

✅ Amazon ECR provides native integration with ECS, IAM-based access control, encrypted image storage, and vulnerability scanning, making it more secure and production-ready compared to public registries.

---

## ✅ Outcome

* Private ECR repository created
* Image scanning enabled
* Repository ready for Docker image push
* Secure container image storage configured

---


