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

## 2.2 IAM Role for CD Server EKS Access

An IAM role is created for the CD server so that the EC2 instance can authenticate with Amazon EKS.

### IAM Role

Create an IAM role with:

- **Role name:** `CD-mainserver-role`
- **Trusted entity:** AWS service
- **Use case:** EC2

An inline IAM policy is added to allow the CD server to describe the EKS cluster.

### Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:DescribeCluster"
      ],
      "Resource": "*"
    }
  ]
}
```

The inline policy is named:

```text
EKSDescribeCluster
```

<img src="images/cd-eks-iam-role.png" width="500">

The `CD-mainserver-role` IAM role is then attached to the `CD-mainserver` EC2 instance.

---

## 2.3 EKS Access Entry

The CD server IAM role is added to the EKS cluster using an **EKS Access Entry**.

Go to:

**EKS → Clusters → chat-cluster → Access → Create access entry**

### Configure IAM Access Entry

- **IAM principal ARN:** `CD-mainserver-role`
- **Type:** `Standard`
- **Kubernetes groups:** Leave empty

<img src="images/eks-access-entry.png" width="500">

### Add Access Policy

Associate the following EKS access policy:

- **Policy:** `AmazonEKSClusterAdminPolicy`
- **Access scope:** `Cluster`

This provides the CD server with cluster administrator access through the EKS API.

<img src="images/eks-access-entry-policy.png" width="500">

After creating the access entry, the CD server can authenticate to the EKS cluster using the IAM role attached to the EC2 instance.

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

[AWS CLI Installation Guide](https://github.com/NithinGowda46/installation_guide/blob/a05684a5e9f54c29c06bbf0135a8ac3900ef1c1f/8.AWS/.installation_amazonlinux.md)

### kubectl

[kubectl Installation Guide](https://github.com/NithinGowda46/installation_guide/blob/a05684a5e9f54c29c06bbf0135a8ac3900ef1c1f/4.kubernetes/kubeinstallation_AWS/kubectl.md)

### Argo CD

[Argo CD Installation Guide](https://github.com/NithinGowda46/installation_guide/blob/a05684a5e9f54c29c06bbf0135a8ac3900ef1c1f/5.agrocd/.installation_amazonlinux.md)

---