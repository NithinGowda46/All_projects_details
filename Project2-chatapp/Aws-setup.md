# DevOps Project

This project demonstrates the implementation of a complete DevOps environment on AWS, including infrastructure, CI/CD, containerization, Kubernetes, monitoring, security, and application deployment.

---

# 1. VPC Setup

## AWS Region

- **Region:** `ap-southeast-1` (Singapore)

## VPC

<img src="images/vpc.png" width="600">

## Subnets

<img src="images/subnets.png" width="600">

## Internet Gateway

- Create an **Internet Gateway** and attach it to the VPC.

## Route Tables

### Public Route Table

<img src="images/public-route-subnet-association.png" width="600">

<img src="images/public-route-routes.png" width="600">

### NAT Gateway

- Create a **NAT Gateway** in a public subnet.

### Private Route Table

<img src="images/private-route-subnet-association.png" width="600">

<img src="images/private-route-routes.png" width="600">