# DevOps Project

This project demonstrates the implementation of a complete DevOps environment on AWS, including infrastructure, CI/CD, containerization, Kubernetes, monitoring, security, and application deployment.

---

# 1. VPC Setup

## AWS Region

- **Region:** `ap-southeast-1` (Singapore)

## VPC

<img src="images/vpc.png" width="450">

## Subnets

<img src="images/subnets.png" width="450">

## Internet Gateway

- Create an **Internet Gateway** and attach it to the VPC.

## Route Tables

### Public Route Table

**Subnet Association**

<img src="images/public-route-subnet-association.png" width="400">

**Routes**

<img src="images/public-route-routes.png" width="400">

### NAT Gateway

- Create a **NAT Gateway** in a public subnet.

### Private Route Table

**Subnet Association**

<img src="images/private-route-subnet-association.png" width="400">

**Routes**

<img src="images/private-route-routes.png" width="400">