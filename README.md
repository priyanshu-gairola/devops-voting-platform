# End-to-End DevOps Platform on AWS EKS

Our repository represents the DevOps platform, not the upstream application.

Terraform → infrastructure
Kubernetes → workload configuration
Helm → application packaging/deployment
Jenkins → CI/CD
Monitoring → observability
Docs → architecture/troubleshooting/interview evidence

## Project Overview

## Architecture

Developer
   ↓
GitHub
   ↓
Jenkins
   ↓
Docker
   ↓
ECR
   ↓
EKS
   ↓
Kubernetes + Helm
   ↓
Voting Application
   ↓
Prometheus + Grafana

Terraform → provisions/manages AWS infrastructure


## Technology Stack

## Repository Structure

## Project Status

## Project Status

### Completed

- Day 1: Project foundation and architecture
- Day 2: Docker containerization and Amazon ECR
  - Built and tested the voting application container
  - Verified Docker networking and service discovery
  - Verified health checks and container lifecycle
  - Created Amazon ECR repository
  - Authenticated Docker with ECR
  - Pushed versioned image `1.0.0` to ECR

### Current Container Artifact

ECR image:

687633314339.dkr.ecr.ap-south-1.amazonaws.com/devops-voting/vote:1.0.0

The image will be consumed later by Kubernetes/EKS.