# 07 – IAM Roles & ECS Cluster (Ritual Roast)

This document explains the IAM roles required for ECS Fargate and the creation of the ECS cluster for the Ritual Roast project.

Proper IAM separation is critical for security and is a common interview topic.

---

## 🧠 Important Concept (Interview Gold)

ECS uses **two different IAM roles**:

| Role Type      | Used By               | Purpose                                            |
| -------------- | --------------------- | -------------------------------------------------- |
| Execution Role | ECS Service           | Pull images from ECR, write logs to CloudWatch     |
| Task Role      | Application Container | Access AWS services like Secrets Manager, S3, etc. |

👉 Never hardcode credentials inside the container.

---

# 1️⃣ Create IAM Policy (Secrets Manager Access)

Navigate to:

**AWS Console → IAM → Policies → Create policy**

---

## Policy Configuration

### Service

* AWS Secrets Manager

### Actions

* Read → `GetSecretValue`

### Resources

Select **Specific ARN** and add the ARN of:

`ritual-roast-db-secret`

Example ARN format:

```
arn:aws:secretsmanager:ap-south-1:<account-id>:secret:ritual-roast-db-secret*
```

---

## Policy Name

| Field       | Value                               |
| ----------- | ----------------------------------- |
| Policy name | ritual-roast-allow-db-secret-policy |

Click **Create policy**.

✅ Custom policy created.

---

# 2️⃣ Create IAM Role for ECS Task

Navigate to:

**AWS Console → IAM → Roles → Create role**

---

## Trusted Entity

| Field          | Value                          |
| -------------- | ------------------------------ |
| Trusted entity | AWS service                    |
| Service        | Elastic Container Service      |
| Use case       | Elastic Container Service Task |

---

## Attach Policies

Attach both policies:

1️⃣ **AWSECSTaskExecutionRolePolicy**
Allows ECS to:

* Pull images from ECR
* Send logs to CloudWatch

2️⃣ **ritual-roast-allow-db-secret-policy**
Allows container to:

* Retrieve DB credentials from Secrets Manager

---

## Role Name

| Field     | Value                      |
| --------- | -------------------------- |
| Role name | ritual-roast-ecs-task-role |

Click **Create role**.

✅ ECS Task Role ready.

---

## 🔐 IAM Flow Explained

```
ECS Service
     |
     |-- Execution Role → ECR + CloudWatch
     |
     |-- Task Role → Secrets Manager → RDS Credentials
```

This ensures:

* No hardcoded secrets
* Least privilege access
* Secure runtime credential fetching

---

# 3️⃣ Create ECS Cluster (Fargate)

Navigate to:

**AWS Console → Amazon ECS → Clusters → Create cluster**

---

## Cluster Configuration

| Field          | Value                    |
| -------------- | ------------------------ |
| Cluster name   | ritual-roast-ecs-cluster |
| Infrastructure | AWS Fargate (Serverless) |

✔ No EC2 instances to manage
✔ No AMI maintenance
✔ Fully managed compute

Click **Create**.

---

## 4️⃣ Verify Cluster

After creation:

* Status: Active
* Capacity providers: FARGATE, FARGATE_SPOT

Cluster is now ready for Task Definition and Service deployment.

---

## 🧠 Interview-Ready Explanation

**Q: Why separate Execution Role and Task Role?**

✅ The Execution Role is used by ECS to manage infrastructure operations like pulling images and pushing logs. The Task Role is assumed by the container at runtime to securely access AWS services like Secrets Manager. This separation follows the principle of least privilege.

---

## ✅ Outcome

* Custom IAM policy created
* Secure ECS Task Role configured
* Proper IAM separation implemented
* Fargate cluster created
* Infrastructure ready for service deployment

---

