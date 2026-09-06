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