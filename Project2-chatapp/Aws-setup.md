# DevOps Dev Environment Implementation

## 1. Infrastructure Setup

### 1.1 AWS Region

- **Region:** `ap-southeast-1` (Singapore)

### 1.2 VPC

![VPC](images/vpc.png)

### 1.3 Subnets

![VPC Subnets](images/subnets.png)

### 1.4 Route Tables

_To be added._

### 1.5 Internet Gateway

_To be added._

### 1.6 NAT Gateway

_To be added._

### 1.7 VPC Endpoints

_To be added._

---

## 2. Server 1 — Jenkins / CI Server

### Purpose

_To be added._

### EC2 Configuration

_To be added._

### Installation

_To be added._

### Configuration

_To be added._

---

## 3. Server 2 — CD / Monitoring Server

### Purpose

_To be added._

### EC2 Configuration

_To be added._

### Installation

_To be added._

### Configuration

_To be added._

---

## 4. EKS Cluster

### Cluster Configuration

_To be added._

### Worker Nodes

- Node 1
- Node 2
- Node 3

### Kubernetes Configuration

_To be added._

---

## 5. Amazon RDS

_To be added._

---

## 6. Amazon ECR

_To be added._

---

## 7. AWS Secrets Manager

_To be added._

---

## 8. IAM Roles and Permissions

_To be added._

---

## 9. Application Deployment

_To be added._

---

## 10. Application Load Balancer / Ingress

_To be added._

---

## 11. CloudFront

_To be added._

---

## 12. AWS WAF

_To be added._

---

## 13. VPN Access

_To be added._

---

## 14. Monitoring

### Prometheus

_To be added._

### Grafana

_To be added._

---

## 15. CI/CD Flow

```text
Developer
    |
    v
Git Repository
    |
    v
Jenkins
    |
    +--> Build
    +--> Test
    +--> SonarQube
    +--> Trivy
    |
    v
Amazon ECR
    |
    v
Argo CD
    |
    v
EKS
    |
    v
Application