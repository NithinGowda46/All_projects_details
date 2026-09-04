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

The public route table allows resources in public subnets to communicate with the Internet.  
An Internet Gateway is used as the Internet-facing route.

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

The private route table is used by resources that should not have direct Internet access.  
A NAT Gateway provides controlled outbound Internet access for private resources.

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