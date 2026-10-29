  # Day 23: AWS IAM & Access Management

An operational guide and architecture reference for Identity and Access Management (IAM) on AWS, created for Day 23 of the DevOps Winter Arc Challenge.

---

## 🎯 Core Objectives

* Establish secure account governance following AWS best practices.
* Implement the principle of least privilege across team roles.
* Design a modular permission structure for users, groups, and service roles.

---

## 🏗️ Architectural Overview

```text
               +---------------------------------------------------+
               |                 AWS ACCOUNT (ROOT)                |
               |             [MFA Enabled | No Daily Use]          |
               +---------------------------------------------------+
                                         |
                       +-----------------+-----------------+
                       |                                   |
              [ IAM User Groups ]                   [ IAM Roles ]
                       |                                   |
        +--------------+--------------+          +---------+---------+
        |              |              |          |                   |
   [ Admins ]    [ Developers ]  [ Auditors ]  [ EC2 Workloads ]  [ Cross-Account ]
        |              |              |          |                   |
   (Full Access)  (Scoped S3)   (Read-Only)  (STS Temp Keys)     (Trust Policy)
```

---

## 🔑 IAM Core Components

* **IAM User:** Identity assigned to a person or application requiring access.
* **IAM Group:** Collection of users sharing common access permissions.
* **IAM Role:** Identity assumed by services or users to obtain temporary security credentials.
* **IAM Policy:** JSON document defining allowed or denied `Actions` on specific `Resources`.
* **Least Privilege:** Security principle granting only permissions required to perform a task.

---

## 📊 Team Access Matrix

| Role / Group | Permitted Actions | Restricted Actions | Primary Use Case |
| :--- | :--- | :--- | :--- |
| **Admins** | Full administrative rights | Direct root user login | Day-to-day cloud management |
| **Developers** | Read-Only baseline + Scoped S3 Read/Write | Account-wide admin, IAM user creation | Application development & storage access |
| **Auditors** | Global Read-Only (`Get*`, `List*`, `Describe*`) | Write, Update, Delete, or Create actions | Compliance & security auditing |

---

## 📄 Policy Blueprint (Developer Access)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ObjectAccess",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::app-lab-bucket/*"
    },
    {
      "Sid": "BucketAccess",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::app-lab-bucket"
    }
  ]
}
```

*Note: Object-level operations (`GetObject`, `PutObject`) target `/*`, whereas bucket-level operations (`ListBucket`) target the bucket ARN directly.*

---

## 💡 Key Design Principles

* **Roles over Static Credentials:** EC2 instances use attached IAM Roles instead of stored access keys. AWS STS automatically provisions and rotates temporary credentials.
* **Restricted Admin Rights:** Developers are excluded from `AdministratorAccess` to minimize blast radius and prevent accidental resource modification or deletion.
* **Default Deny Model:** All requests are denied by default unless explicitly allowed by an attached policy.

---

## 💻 CLI Identity Diagnostics

```bash
# Verify CLI configuration version
aws --version

# Confirm active caller identity and target account
aws sts get-caller-identity

# Check active profile settings
aws configure list

# Query account summary statistics
aws iam get-account-summary
```

---

## ❓ Troubleshooting Common Issues

* **Cannot create IAM user:** Missing `iam:CreateUser` permissions on active identity.
* **Can read object but cannot list bucket:** `s3:GetObject` applies to objects (`/*`), whereas `s3:ListBucket` is required on the bucket ARN.
* **Role assumption failure:** Misconfigured Trust Policy or missing `sts:AssumeRole` permissions.
* **Policy flag (`Action: *`, `Resource: *`):** Violates least privilege; scope down to specific actions and resource ARNs.

---

## ✅ Completion Checklist

- [x] Multi-Factor Authentication (MFA) enabled on root user.
- [x] Budget alerts configured for $0 threshold monitoring.
- [x] Team access matrix designed (Admins, Developers, Auditors).
- [x] IAM policies scoped to specific resources.
- [x] Verified zero chargeable AWS resources were launched.