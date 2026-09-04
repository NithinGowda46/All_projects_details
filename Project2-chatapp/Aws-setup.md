# DevOps Project

This project demonstrates the implementation of a complete DevOps environment on AWS, including infrastructure, CI/CD, containerization, Kubernetes, monitoring, security, and application deployment.

---

# 1. VPC Setup

## 1.1 VPC

### AWS Region

- **Region:** `ap-southeast-1` (Singapore)

### VPC

<img src="images/vpc.png" width="400">

### Subnets

<img src="images/subnets.png" width="400">

---

## 1.2 Public Route Table

### Internet Gateway

- Create an **Internet Gateway** and attach it to the VPC.

### Subnet Association

<img src="images/public-route-subnet-association.png" width="350">

### Routes

<img src="images/public-route-routes.png" width="350">

---

## 1.3 Private Route Table

### NAT Gateway

- Create a **NAT Gateway** in a public subnet.

### Subnet Association

<img src="images/private-route-subnet-association.png" width="350">

### Routes

<img src="images/private-route-routes.png" width="350">