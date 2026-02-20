# 03 – Docker Image Build & Push to ECR (Ritual Roast)

This document explains how to **build a Docker image** for the Ritual Roast application and **push it securely to Amazon ECR** using a **private EC2 build server** with **Session Manager (no SSH)**.

---

## 🎯 Objective

* Create an IAM role for a secure EC2 build server
* Launch a private EC2 instance (no public IP)
* Install Docker and AWS CLI
* Build the Docker image
* Push the image to Amazon ECR

---

## 1️⃣ Create IAM Role for EC2 (Docker + ECR + SSM)

Navigate to:

**AWS Console → IAM → Roles → Create role**

### Trusted Entity

| Field          | Value       |
| -------------- | ----------- |
| Trusted entity | AWS service |
| Use case       | EC2         |

### Attach Permissions

Attach the following **AWS-managed policies**:

1. **AmazonSSMManagedInstanceCore**

   * Enables Session Manager access (no SSH, no keys)

2. **EC2InstanceProfileForImageBuilderECRContainerBuilds**

   * Allows Docker image push to Amazon ECR

### Role Name

| Field     | Value                            |
| --------- | -------------------------------- |
| Role name | iam-grant-ec2-ssm-and-ecr-access |

Click **Create role**.

✅ IAM role created successfully.

---

## 2️⃣ Launch EC2 Docker Build Server

Navigate to:

**AWS Console → EC2 → Launch instance**

### Basic Configuration

| Field         | Value                            |
| ------------- | -------------------------------- |
| Name          | ritual-roast-docker-build-server |
| AMI           | Ubuntu Server 22.04 LTS          |
| Instance type | t2.micro (lab)                   |
| Key pair      | None                             |

### Networking

| Field                 | Value                    |
| --------------------- | ------------------------ |
| VPC                   | Ritual-roast-vpc         |
| Subnet                | rr-private-app-subnet-1a |
| Auto-assign Public IP | Disabled                 |
| Security group        | Default                  |

✔ Instance is fully private

### IAM Role

| Field    | Value                            |
| -------- | -------------------------------- |
| IAM role | iam-grant-ec2-ssm-and-ecr-access |

Click **Launch instance**.

---

## 3️⃣ Connect Using Session Manager (No SSH)

Wait until:

* Instance state = **Running**
* SSM status = **Online**

### Connection Steps

1. EC2 → Select instance
2. Click **Connect**
3. Choose **Session Manager**
4. Click **Connect**

✅ Secure access without:

* SSH keys
* Open ports
* Bastion hosts

---

## 4️⃣ Install Docker on Ubuntu

Run the following commands inside Session Manager:

```bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
newgrp docker
docker --version
```

✅ Docker installed successfully.

---

## 5️⃣ Install AWS CLI

```bash
sudo apt install awscli -y
aws --version
```

📌 Authentication is handled automatically using the EC2 IAM role (no access keys).

---

## 6️⃣ Clone Application Source Code

```bash
sudo apt install unzip git -y
git clone https://github.com/iaasacademy/aws-how-to-guide.git
cd aws-how-to-guide/amazon-ecs-mini-project
unzip ritual-roast-code.zip
cd ritual-roast-code
```

---

## 7️⃣ Build Docker Image

### Build Command

```bash
docker build -t <ECR-REPOSITORY-URI>:latest .
```

### Verify Image

```bash
docker images
```

---

## 8️⃣ Authenticate & Push Image to ECR

Go to **ECR → ritual-roast repository → View push commands** and run the provided commands.

Typical flow:

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
docker push <ECR-REPOSITORY-URI>:latest
```

✅ Docker image pushed successfully.

---

## 🧠 Interview-Ready Explanation

**Q: Why use a private EC2 build server instead of local machine?**

✅ A private EC2 build server improves security, avoids credential exposure, integrates with IAM roles, and uses Session Manager instead of SSH, following AWS best practices.

---

## ✅ Outcome

* Secure EC2-based Docker build server
* No SSH or access keys used
* Docker image built successfully
* Image pushed to Amazon ECR
* Ready for ECS task definition

---

