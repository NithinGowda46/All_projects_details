# DevOps Project

This project demonstrates the implementation of a complete DevOps environment on AWS, including infrastructure, CI/CD, containerization, Kubernetes, monitoring, security, and application deployment.

---

# 1. VPC Setup

## AWS Region

- **Region:** `ap-southeast-1` (Singapore)

## VPC

![VPC](images/vpc.png)

## Subnets

![VPC Subnets](images/subnets.png)

## Internet Gateway

- Create an **Internet Gateway** and attach it to the VPC.

## Route Tables

### Public Route Table

![Public Route Table - Subnet Association](images/public-route-subnet-association.png)

![Public Route Table - Routes](images/public-route-routes.png)

### NAT Gateway

- Create a **NAT Gateway** in a public subnet.

### Private Route Table

![Private Route Table - Subnet Association](images/private-route-subnet-association.png)

![Private Route Table - Routes](images/private-route-routes.png)