# 🔴 CD Setup

This section covers the resources required for Continuous Deployment and application deployment.

---

# 🔴 1. Amazon RDS

Amazon RDS is used as the managed PostgreSQL database for the application.  
The database is deployed inside the VPC with private access.  
AWS Secrets Manager is used to manage the database credentials.

<img src="images/rds.png" width="500">

---

# 🔴 2. Amazon EKS

Amazon EKS is used to run the application containers using Kubernetes.

## 2.1 EKS Cluster Creation

Create the EKS cluster with **EKS Auto Mode enabled**.

After reviewing the configuration, select **Create** to create the EKS cluster.

> No separate node group is required because EKS Auto Mode automatically manages the cluster compute resources.

---

# 🔴 3. CD Server Setup

The CD server is used for Continuous Deployment and deployment-related operations.

- **Name:** `CD-mainserver`
- **Instance Type:** `m7i-flex.large`
- **vCPU:** 2
- **Memory:** 8 GB
- **Subnet:** `1b_private_subnet`
- **Private IP:** `10.18.35.188`
- **Public IPv4:** None
- **Operating System:** Amazon Linux 2023
- **IMDSv2:** Required

<img src="images/cd-server.png" width="350">

## 3.2 Install Required Tools

### AWS CLI

[AWS CLI Installation Guide](#)

### kubectl

[kubectl Installation Guide](#)

### Argo CD

[Argo CD Installation Guide](#)


---