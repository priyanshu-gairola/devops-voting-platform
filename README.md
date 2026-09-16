# DevOps Voting Platform

A production-oriented DevOps project built around Docker's Example Voting App.

The purpose of this project is to build and operate a complete application platform while learning DevOps and Cloud concepts through hands-on implementation, troubleshooting, failure testing, and recovery.

## Project Goal

Build an end-to-end DevOps platform covering:

**CODE → GIT → CI → DOCKER → REGISTRY → TERRAFORM → AWS → EKS → KUBERNETES → HELM → CD → APPLICATION → MONITORING → SECURITY → TROUBLESHOOTING → RECOVERY**

The focus is primarily on DevOps, Cloud, Infrastructure and Operations, while keeping application development limited to what is required to understand and operate the platform.

---

# Architecture

Current architecture developed through Day 8:

```text
Developer
   |
   | Git Push
   v
GitHub
   |
   | CI
   v
GitHub Actions
   |
   | Build Docker Image
   v
Amazon ECR
   |
   v
Amazon EKS
   |
   +----------------------+
   |                      |
   v                      v
ALB                 Kubernetes
   |                 Ingress
   |                      |
   +-----------> Services
                         |
              +----------+----------+
              |          |          |
             Vote      Result     Worker
              |          |          |
             Redis <---- Worker    PostgreSQL
                         |
                       Result
```

---

# Technology Stack

* Git / GitHub
* GitHub Actions
* Docker
* Docker Compose
* Amazon ECR
* AWS
* Terraform
* Amazon EKS
* Kubernetes
* Helm
* AWS Load Balancer Controller
* Application Load Balancer
* Redis
* PostgreSQL
* Python / Flask
* Node.js
* .NET

---

# Project Structure

```text
devops-voting-platform/
│
├── app/
│
├── kubernetes/
│   ├── db-deployment.yaml
│   ├── db-pvc.yaml
│   ├── db-secret.yaml
│   ├── db-service.yaml
│   ├── redis-deployment.yaml
│   ├── redis-service.yaml
│   ├── result-deployment.yaml
│   ├── result-service.yaml
│   ├── vote-deployment.yaml
│   ├── vote-service.yaml
│   ├── worker-deployment.yaml
│   └── ingress.yaml
│
├── helm/
│   └── voting-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│
├── terraform/
│
├── .github/
│   └── workflows/
│
└── README.md
```

---

# Day 1 — Project Foundation

## Objective

Understand the application and establish the Git-based project workflow.

### Covered

* Understanding the Example Voting App architecture
* Application components
* Git repository setup
* Branching basics
* Main branch as the production branch
* Meaningful commits
* Basic DevOps project structure

### Application Components

```text
Vote Service
    |
    v
Redis
    |
    v
Worker
    |
    v
PostgreSQL
    |
    v
Result Service
```

The application consists of:

* Vote service
* Redis
* Worker
* PostgreSQL
* Result service

---

# Day 2 — Docker

## Objective

Containerize and understand the application components.

### Covered

* Docker images
* Docker containers
* Image layers
* Dockerfiles
* `.dockerignore`
* Multi-stage builds
* Docker Compose
* Container networking
* Environment variables
* Container logs
* Container troubleshooting

### Important Concepts

**Image**

A packaged application filesystem and configuration used to create containers.

**Container**

A running instance of an image.

**Docker Compose**

Used to run multiple related containers as an application stack.

Example:

```bash
docker compose up -d
```

Verification:

```bash
docker ps
```

Logs:

```bash
docker logs <container>
```

---

# Day 3 — Git & CI

## Objective

Introduce continuous integration and establish an automated image build pipeline.

### Covered

* Git branching
* `main` as production branch
* `git revert`
* `git reset`
* GitHub Actions
* Automated Docker builds
* Commit SHA image tagging
* Docker image publishing

The CI pipeline performs:

```text
Git Push
   |
   v
GitHub Actions
   |
   v
Checkout Code
   |
   v
Get Commit SHA
   |
   v
Build Docker Image
   |
   +----> latest
   |
   +----> commit SHA
   |
   v
Push Image
```

### Why SHA Tagging?

Instead of relying only on:

```text
latest
```

we also use a unique commit-based tag.

This allows us to identify exactly which source version produced an image.

---

# Day 4 — AWS & Container Registry

## Objective

Move the container workflow toward AWS.

### Covered

* AWS fundamentals
* EC2
* AMI
* Instance types
* Key pairs
* Security Groups
* Elastic IP
* Amazon ECR
* IAM roles
* Load Balancer
* Target Groups
* Auto Scaling Groups
* Launch Templates

### Application Architecture

```text
Internet
   |
   v
ALB
   |
   v
Target Group
   |
   v
EC2
   |
   v
Docker Container
   |
   v
Application
```

ECR became the container image registry.

---

# Day 5 — Terraform

## Objective

Start managing AWS infrastructure as code.

### Covered

* Infrastructure as Code
* Terraform configuration
* Providers
* Resources
* Variables
* Terraform state
* `terraform init`
* `terraform plan`
* `terraform apply`
* `terraform destroy`

Terraform was used for AWS infrastructure including ECR and networking/security resources.

### Terraform Workflow

```text
Terraform Configuration
        |
        v
terraform init
        |
        v
terraform plan
        |
        v
terraform apply
        |
        v
AWS Infrastructure
```

### Cost Control

AWS resources are destroyed after practical sessions whenever they are no longer required.

```bash
terraform destroy
```

This is an important part of responsible Cloud operations.

---

# Day 6 — Amazon EKS & Kubernetes Foundation

## Objective

Move the application from individual EC2/container management to Kubernetes.

### Covered

* Kubernetes fundamentals
* EKS
* Nodes
* Pods
* Deployments
* Services
* Persistent Volumes
* Persistent Volume Claims
* Kubernetes networking
* Application deployment
* `kubectl`
* EKS authentication

### Basic Kubernetes Architecture

```text
EKS Cluster
   |
   +--- Node
   |      |
   |      +--- Pod
   |      +--- Pod
   |
   +--- Node
          |
          +--- Pod
```

### Verification

```bash
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl get services
```

### EKS Authentication

When Kubernetes authentication failed, the kubeconfig was refreshed using:

```bash
aws eks update-kubeconfig
```

and the EKS token mechanism was verified.

---

# Day 7 — Kubernetes Networking & AWS ALB

## Objective

Expose the Kubernetes application externally using an AWS Application Load Balancer.

## Architecture

```text
Internet
   |
   v
AWS ALB
   |
   v
Kubernetes Ingress
   |
   v
Kubernetes Service
   |
   v
Pod
```

### Covered

* Kubernetes Services
* Ingress
* IngressClass
* AWS Load Balancer Controller
* AWS ALB
* Target type `ip`
* Internet-facing ALB
* EKS Pod Identity
* IAM permissions
* Terraform-managed IAM configuration

### Ingress

The application used an ALB Ingre
