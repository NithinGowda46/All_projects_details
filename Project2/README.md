# Project 2 — Production DevOps Chat Application

A production-oriented DevOps implementation for a containerized chat application, built on AWS with automated CI/CD, Kubernetes, GitOps, managed data services, secrets management, observability, security controls, and public HTTPS delivery.

> **Portfolio focus:** This project demonstrates how application code moves from GitHub through Jenkins CI, Amazon ECR, Argo CD, and Amazon EKS, while the application is exposed through CloudFront and an AWS Application Load Balancer and backed by Amazon DocumentDB.

## Architecture at a Glance

```text
                         INTERNET
                            │
                         HTTPS :443
                            │
                       CloudFront
                            │
                       HTTP :80
                            │
                  Internet-facing ALB
                            │
                    Amazon EKS Auto Mode
                     ┌──────┴──────┐
                     │             │
                 Frontend      Backend API
                                   │
                          ┌────────┴────────┐
                          │                 │
                    DocumentDB       AWS Secrets Manager

Developer
   │
   ▼
GitHub CI Repository
   │
   ▼
Jenkins (Private CI Server)
   │
   ├── Build frontend/backend images
   ├── Push images to Amazon ECR
   └── Update image tags in CD repository
                         │
                         ▼
                 GitHub CD Repository
                         │
                         ▼
                      Argo CD
                         │
                         ▼
                 Amazon EKS Auto Mode

Private administration:
AWS Client VPN → private CI/CD infrastructure → Argo CD / Grafana
```

## What This Project Demonstrates

- AWS VPC design with public and private subnets
- Secure private access using AWS Client VPN
- Jenkins-based continuous integration
- Docker image creation and Amazon ECR image storage
- Kubernetes deployment on Amazon EKS Auto Mode
- GitOps-based continuous delivery with Argo CD
- Separate CI and CD repositories
- Amazon DocumentDB integration for application data
- AWS Secrets Manager with EKS Pod Identity and Secrets Store CSI Driver
- Application exposure through an internet-facing ALB
- CloudFront-based HTTPS delivery and Route 53 DNS
- AWS WAF, security groups, and least-privilege IAM controls
- Kubernetes Horizontal Pod Autoscaler (HPA)
- CloudWatch, Prometheus, and Grafana monitoring
- Private administration of DevOps tooling through the VPN

## Technology Stack

| Area | Technology |
|---|---|
| Cloud | AWS |
| Region | `ap-southeast-1` (Singapore) |
| Source Control | GitHub |
| CI | Jenkins |
| Containers | Docker |
| Image Registry | Amazon ECR |
| Orchestration | Amazon EKS Auto Mode |
| Kubernetes Delivery | Argo CD |
| Database | Amazon DocumentDB |
| Secrets | AWS Secrets Manager + EKS Pod Identity + Secrets Store CSI Driver |
| DNS | Amazon Route 53 |
| CDN / HTTPS | Amazon CloudFront + ACM |
| Edge Security | AWS WAF |
| Monitoring | CloudWatch + Prometheus + Grafana |
| Autoscaling | Kubernetes HPA |
| Private Access | AWS Client VPN |

## CI/CD Workflow

### Continuous Integration

1. Developer pushes application changes to the CI repository.
2. Jenkins detects the change and starts the pipeline.
3. Jenkins builds the frontend and backend Docker images.
4. Images are tagged and pushed to Amazon ECR.
5. The pipeline updates the image tags in the CD repository.

### Continuous Delivery

1. Argo CD watches the CD repository.
2. A new image tag in Git becomes the desired deployment state.
3. Argo CD synchronizes Kubernetes manifests to Amazon EKS.
4. Kubernetes performs the rolling deployment.
5. HPA can scale application workloads according to configured resource usage.

This separates **build responsibility** from **deployment responsibility** and keeps Git as the source of truth for the Kubernetes application state.

## Security & Secrets Flow

Application secrets are not stored as Kubernetes manifests or hard-coded into application images.

```text
AWS Secrets Manager
        │
        │ chatapp/backend
        ▼
Secrets Store CSI Driver / ASCP
        │
        ▼
Kubernetes Secret: backend-secret
        │
        ▼
Backend Deployment
        │
        └── MongoDB URI / JWT secret

EKS Pod Identity
        │
        ▼
ChatAppBackendSecretsRole
```

The backend workload uses a dedicated Kubernetes service account and IAM role for accessing the required AWS secret. Static AWS access keys are not required inside the Kubernetes workload.

## Production Traffic Flow

```text
User Browser
    │
    │ HTTPS
    ▼
CloudFront
    │
    │ HTTP :80 to origin
    ▼
EKS Application Load Balancer
    │
    ▼
Frontend / Backend Services
    │
    ▼
Amazon DocumentDB
```

The public domain is managed through Route 53 and secured at the viewer edge using an ACM certificate associated with CloudFront. The current origin configuration uses **HTTP :80 between CloudFront and the ALB**.

## Infrastructure & Access Model

The project uses a VPC with public and private subnets.

- **Public layer:** internet-facing ALB and required public AWS networking components.
- **Private layer:** CI/CD EC2 servers and internal administration paths.
- **VPN access:** AWS Client VPN provides controlled access to private infrastructure.
- **CI server:** Jenkins and Docker build tooling run on the private CI server.
- **CD server:** deployment administration and Kubernetes/Argo CD tooling are managed from the private CD server.
- **Monitoring:** Grafana and Argo CD administration are intended to remain privately accessible rather than permanently exposing management interfaces to the internet.

> **Practice note:** Jenkins `8080` is exposed through the ALB in this project to make the CI environment accessible during practice/testing. This is **not recommended for a production Jenkins deployment**; production environments should prefer private access through VPN or another controlled access mechanism.

## Repository Structure

| File / Directory | Purpose |
|---|---|
| [`1.CI-setup.md`](1.CI-setup.md) | CI infrastructure, Jenkins, Docker, ECR, and pipeline setup |
| [`2.CD-setup.md`](2.CD-setup.md) | DocumentDB, EKS, Kubernetes, secrets, and Argo CD setup |
| [`3.final-CI-CD.md`](3.final-CI-CD.md) | Final Jenkins-to-GitOps CI/CD workflow |
| [`4.production_setup.md`](4.production_setup.md) | Route 53, ACM, CloudFront, ALB, and production traffic |
| [`5.monitoring_setup.md`](5.monitoring_setup.md) | CloudWatch, Prometheus, Grafana, and monitoring access |
| [`6.security_setup.md`](6.security_setup.md) | Security groups and security architecture |
| [`deployment_prerequisites.md`](deployment_prerequisites.md) | EKS prerequisites, secrets integration, and Pod Identity |
| [`k8s_explain/`](k8s_explain/) | Kubernetes resource explanations and manifests |
| [`images/`](images/) | Architecture, infrastructure, CI/CD, and monitoring screenshots |

## Recommended Reading Order

**01 — CI foundation**  
Start with [`1.CI-setup.md`](1.CI-setup.md) to understand the VPC, private CI server, Jenkins, Docker, ECR, and image build process.

**02 — CD and Kubernetes**  
Continue with [`2.CD-setup.md`](2.CD-setup.md) for EKS, DocumentDB, secrets integration, Kubernetes resources, and Argo CD.

**03 — End-to-end CI/CD**  
Read [`3.final-CI-CD.md`](3.final-CI-CD.md) to see how Jenkins publishes images and updates the GitOps repository.

**04 — Production delivery**  
Read [`4.production_setup.md`](4.production_setup.md) for CloudFront, Route 53, ACM, ALB, and the public application path.

**05 — Observability**  
Read [`5.monitoring_setup.md`](5.monitoring_setup.md) for CloudWatch, Prometheus, and Grafana.

**06 — Security**  
Finish with [`6.security_setup.md`](6.security_setup.md) for network security groups and access boundaries.

## Related Repositories

- **CI repository:** [Project2-chatapp-CI](https://github.com/NithinGowda46/Project2-chatapp-CI) — application source, Dockerfiles, and Jenkins pipeline.
- **CD repository:** [Project2-chatapp-CD](https://github.com/NithinGowda46/Project2-chatapp-CD) — Kubernetes manifests and GitOps deployment configuration.

## Key Portfolio Takeaways

This project is designed to demonstrate more than individual tool installation. The main objective is to show an end-to-end operating model:

**Code → CI → Container Image → Registry → GitOps → Kubernetes → Production Traffic → Monitoring**

It also demonstrates the separation of application delivery from infrastructure access, the use of managed AWS services where appropriate, and the treatment of secrets and operational tooling as security-sensitive components.

## Notes for Public Portfolio Use

- Replace or remove environment-specific identifiers if this repository is shared publicly.
- Never commit real passwords, private keys, access keys, or sensitive connection strings.
- Treat the Jenkins `8080` public exposure as a learning/practice configuration, not a production recommendation.
- HPA is part of the implementation; VPA is intentionally not included.


👨‍💻 Author

Nithin Gowda

This project demonstrates an end-to-end DevOps workflow covering Docker, Jenkins CI, Amazon ECR, Kubernetes, Amazon EKS, GitOps, Argo CD, Gunicorn, and WSGI, along with AWS networking, secrets management, monitoring, security, and production delivery.
