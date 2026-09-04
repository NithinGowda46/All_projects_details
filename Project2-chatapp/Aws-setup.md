# DevOps Project

This project demonstrates the implementation of a complete DevOps environment on AWS, including CI/CD, containerization, Kubernetes, monitoring, security, and application deployment.

---

# 1. VPC Setup

## 1.1 VPC

The VPC provides an isolated network environment for all AWS resources.  
It contains separate public and private subnets for better security and organization.

### AWS Region

- **Region:** `ap-southeast-1` (Singapore)

<table>
<tr>
<th>VPC</th>
<th>Subnets</th>
</tr>
<tr>
<td>
<img src="images/vpc.png" width="300">
</td>
<td>
<img src="images/subnets.png" width="300">
</td>
</tr>
</table>

---

## 1.2 Public Route Table

The public route table provides Internet connectivity to resources deployed in public subnets.  
An Internet Gateway is attached to the VPC for Internet access.

### Internet Gateway

- Create an **Internet Gateway** and attach it to the VPC.

<table>
<tr>
<th>Subnet Association</th>
<th>Routes</th>
</tr>
<tr>
<td>
<img src="images/public-route-subnet-association.png" width="300">
</td>
<td>
<img src="images/public-route-routes.png" width="300">
</td>
</tr>
</table>

---

## 1.3 Private Route Table

The private route table is used for resources that should not be directly accessible from the Internet.  
A NAT Gateway provides outbound Internet connectivity for resources in private subnets.

### NAT Gateway

- Create a **NAT Gateway** in a public subnet.

<table>
<tr>
<th>Subnet Association</th>
<th>Routes</th>
</tr>
<tr>
<td>
<img src="images/private-route-subnet-association.png" width="300">
</td>
<td>
<img src="images/private-route-routes.png" width="300">
</td>
</tr>
</table>

---

## 1.4 AWS Client VPN

The AWS Client VPN provides secure access from the local machine to resources inside the VPC.  
It allows access to private resources using their private IP addresses.

Refer to the following guide to create and configure the AWS Client VPN:

[Refer to AWS Client VPN Setup Guide](https://github.com/NithinGowda46/installation_guide/blob/fc753c567fac7177054e2c977604a045ce3ff93d/8.AWS/vpn.md)

---

# 2. EC2 Instance Setup

## 2.1 CI Server

The CI server is used for Jenkins and other CI/CD tools.  
It is deployed in the private subnet for secure access.

<img src="images/ci-server.png" width="350">

### Configuration

- **Name:** `CI-mainserver`
- **Instance Type:** `m7i-flex.large`
- **vCPU:** 2
- **Memory:** 8 GB
- **Subnet:** `1a_private_subnet`
- **Public IPv4:** None
- **Private IPv4:** `10.18.27.100`
- **IMDSv2:** Required

---

## 2.2 CD Server

The CD server is used for Argo CD, Prometheus, and Grafana.  
It is deployed in the private subnet for secure access.

<img src="images/cd-server.png" width="350">

### Configuration

- **Name:** `CD-mainserver`
- **Instance Type:** `m7i-flex.large`
- **vCPU:** 2
- **Memory:** 8 GB
- **Subnet:** `1b_private_subnet`
- **Public IPv4:** None
- **Private IPv4:** `10.18.35.188`
- **IMDSv2:** Required