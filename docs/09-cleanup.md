# 09 – Cleanup & Resource Deletion Guide (Ritual Roast)

This document explains how to safely delete all AWS resources created for the Ritual Roast ECS Fargate project.

⚠️ Always delete resources in dependency order to avoid errors and unexpected charges.

---

# 🧹 Deletion Order Overview

Delete from **Top (Application Layer)** → **Bottom (Network Layer)**

```
ECS → ALB → RDS → Secrets → ECR → EC2 → IAM → Networking (VPC last)
```

---

# 1️⃣ Delete ECS Resources (First)

## Step 1: Delete ECS Service

Navigate to:

ECS → Clusters → ritual-roast-ecs-cluster → Services

1. Select `ritual-roast-service`
2. Update desired tasks to **0**
3. Wait until tasks stop
4. Click **Delete service**

✔ This automatically deregisters targets from ALB.

---

## Step 2: Delete ECS Cluster

ECS → Clusters → Delete `ritual-roast-ecs-cluster`

---

## Step 3: Deregister Task Definitions (Optional but Clean)

ECS → Task Definitions → Deregister all versions of:

`ritual-roast-task-definition`

---

# 2️⃣ Delete Load Balancer Resources

## Step 4: Delete Application Load Balancer

EC2 → Load Balancers → Select `ritual-roast-alb` → Delete

---

## Step 5: Delete Target Group

EC2 → Target Groups → Select `ritual-roast-tg` → Delete

---

# 3️⃣ Delete Database & Secrets

## Step 6: Delete RDS Database

RDS → Databases → Select `ritualroastdb` → Delete

For lab purposes:

* Disable final snapshot
* Disable automated backups

⏳ Deletion time: 5–10 minutes

---

## Step 7: Delete DB Subnet Group

RDS → Subnet Groups → Delete `ritual-roast-db-subnet-group`

---

## Step 8: Delete Secrets Manager Secret

Secrets Manager → Select `ritual-roast-db-secret`

* Delete secret
* Optionally force delete immediately

---

# 4️⃣ Delete ECR Repository

ECR → Repositories → Select `ritual-roast`

* Delete repository
* Check “Force delete” to remove images

---

# 5️⃣ Delete EC2 Build Server

EC2 → Instances → Select `ritual-roast-docker-build-server`

* Terminate instance

---

# 6️⃣ Delete IAM Resources

## Step 9: Delete IAM Roles

IAM → Roles → Delete:

* ritual-roast-ecs-task-role
* iam-grant-ec2-ssm-and-ecr-access

⚠️ Detach custom policies if required.

---

## Step 10: Delete Custom IAM Policies

IAM → Policies → Delete:

* ritual-roast-allow-db-secret-policy

(AWS-managed policies cannot be deleted.)

---

# 7️⃣ Delete Networking Resources (VPC Last)

## Step 11: Delete NAT Gateways

VPC → NAT Gateways → Delete:

* rr-natgw-1a
* rr-natgw-1b

⏳ Wait until status becomes **Deleted**

---

## Step 12: Release Elastic IPs

VPC → Elastic IPs → Release associated EIPs

---

## Step 13: Delete Internet Gateway

VPC → Internet Gateways

1. Detach `rr-igw`
2. Delete it

---

## Step 14: Delete Route Tables

Delete the following:

* rr-public-rt
* rr-private-app-rt-1a
* rr-private-app-rt-1b
* rr-private-data-rt-1a
* rr-private-data-rt-1b

---

## Step 15: Delete Subnets

Delete all:

* 2 Public subnets
* 2 Private app subnets
* 2 Private data subnets

---

## Step 16: Delete VPC

Finally delete:

`Ritual-roast-vpc`

🎉 All infrastructure removed.

---

# 💰 Final Cost Verification (Very Important)

Go to:

Billing → Cost Explorer

Check for:

* NAT Gateway charges
* RDS charges
* ALB charges

If daily cost = $0 or ₹0 → Cleanup successful.

---

# 🧠 Interview Tip

**Q: How do you safely clean up AWS infrastructure?**

✅ I delete resources in dependency order: ECS → ALB → RDS → ECR → IAM → Networking (VPC last). This prevents dependency conflicts and ensures no billable resources like NAT Gateways remain active.

---

## ✅ Outcome

* All AWS resources deleted safely
* No orphaned dependencies
* No unexpected billing
* Clean and professional teardown process documented

---

🎯 Project Completed Successfully
