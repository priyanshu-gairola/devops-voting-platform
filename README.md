# DevOps Voting Platform

A production-oriented DevOps platform built around the Docker Example Voting App.

The purpose of this project is not to build the application itself, but to design, provision, deploy, automate, monitor, troubleshoot, and operate the application using modern DevOps and cloud technologies.

---

## Project Objective

Build an end-to-end DevOps platform covering:

- Git / GitHub
- Docker / Docker Compose
- AWS
- Amazon ECR
- Terraform
- Amazon EKS
- Kubernetes
- Helm
- AWS Load Balancer Controller
- Application Load Balancer
- Jenkins CI/CD
- Prometheus
- Grafana
- Alerting
- Reliability
- Troubleshooting
- Failure recovery
- AWS cost management

The project follows a build-first and production-oriented approach.

---

# Architecture

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   +--------------------+
   |                    |
   v                    v
Docker Build          Validation
   |
   v
Amazon ECR
   |
   v
Helm Deployment
   |
   v
Amazon EKS
   |
   +-----------------------------+
   |                             |
   v                             v
Kubernetes                    Monitoring
   |                             |
   v                             v
Voting Application          Prometheus
   |                             |
   v                             v
AWS Load Balancer           Grafana
Controller                      |
   |                             v
   v                         Alert Rules
Application Load                 |
Balancer                         v
   |                         Notifications
   v
Users

REPOSITORY STRUCTURE
=====================
devops-voting-platform/
|
+-- terraform/
|   +-- main.tf
|   +-- providers.tf
|   +-- variables.tf
|   +-- outputs.tf
|   +-- aws-load-balancer-controller-policy.json
|
+-- kubernetes/
|   +-- ...
|
+-- helm/
|   +-- voting-app/
|       +-- Chart.yaml
|       +-- values.yaml
|       +-- templates/
|
+-- jenkins/
|   +-- Jenkinsfile
|
+-- docs/
|   +-- ...
|
+-- README.md