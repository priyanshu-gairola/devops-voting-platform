# End-to-End DevOps Platform on AWS EKS

> **Production-style DevOps project built around the Docker Example Voting App**
>
> Infrastructure as Code • AWS EKS • Kubernetes • Helm • Jenkins CI/CD • Amazon ECR • AWS ALB • Prometheus • Grafana • Reliability • Failure Engineering

---

## 1. Project Overview

This project demonstrates how I designed and operated a production-style DevOps platform for a containerized voting application on AWS.

The application itself is based on the open-source **Docker Example Voting App**. I intentionally did not spend the project on application development. Instead, the focus was on the platform required to build, deploy, expose, monitor, troubleshoot, and recover the application.

The platform covers:

- Infrastructure provisioning with **Terraform**
- AWS networking and **Amazon EKS**
- Container image management with **Amazon ECR**
- Kubernetes workload management
- Application packaging with **Helm**
- External traffic through **AWS Application Load Balancer**
- CI/CD using **Jenkins**
- Monitoring with **Prometheus**
- Visualization and alerting with **Grafana**
- IAM and least-privilege concepts
- Kubernetes Secrets
- Failure injection and troubleshooting
- Rollout and smoke-test verification
- AWS resource cleanup and cost-control discipline

The project was deliberately built around real operational problems rather than only successful deployment scenarios.

---

# 2. What Problem Does This Project Solve?

A containerized application is not production-ready simply because the container starts.

A DevOps platform must answer questions such as:

- How is the infrastructure created consistently?
- Where are container images stored?
- How are application versions deployed?
- How does external traffic reach Kubernetes Pods?
- How are unhealthy workloads detected?
- How are deployments verified?
- How do we monitor infrastructure and workloads?
- What happens when an image cannot be pulled?
- What happens when a container repeatedly crashes?
- What happens when monitoring cannot schedule?
- How do we troubleshoot failures using evidence?
- How do we clean up cloud infrastructure safely?

This project addresses those operational concerns end to end.

---

# 3. High-Level Architecture

```text
                         DEVELOPER / SOURCE
                                |
                              GitHub
                                |
                             Jenkins
                                |
                         Docker Image Build
                                |
                           Amazon ECR
                                |
                              Helm
                                |
                           Amazon EKS
                                |
                    +-----------+-----------+
                    |                       |
              Kubernetes                Monitoring
                    |                       |
                 Ingress                Prometheus
                    |                       |
               AWS ALB                  PromQL
                    |                       |
                 Service                 Grafana
                    |                       |
                   Pods              Dashboards / Alerts
                    |
          +---------+---------+
          |         |         |
         Vote      Result    Worker
          |                    |
        Redis               PostgreSQL
                              |
                         Persistent Storage
```

---

# 4. Architecture Flows

## 4.1 Infrastructure Flow

```text
Terraform
   ↓
AWS
   ↓
VPC / Networking
   ↓
IAM
   ↓
EKS Control Plane
   ↓
EKS Node Group
   ↓
Worker Nodes
```

Terraform is used to define and provision the AWS infrastructure instead of manually creating the environment.

---

## 4.2 Runtime Flow

```text
User
   ↓
AWS Application Load Balancer
   ↓
Kubernetes Ingress
   ↓
Kubernetes Service
   ↓
Pod
   ↓
Container
   ↓
Application
   ↓
Redis / Worker / PostgreSQL
```

The voting workload contains separate services for voting, results, background processing, Redis and PostgreSQL.

A simplified voting path is:

```text
User
 ↓
ALB
 ↓
Ingress
 ↓
Vote Service
 ↓
Vote Pod
 ↓
Voting Application
 ↓
Redis
 ↓
Worker
 ↓
PostgreSQL
```

The result application reads persisted results from PostgreSQL.

---

## 4.3 Developer / CI-CD Flow

```text
Developer
   ↓
GitHub
   ↓
Jenkins
   ↓
Docker Build
   ↓
Docker Image
   ↓
Amazon ECR
   ↓
Helm
   ↓
Kubernetes / EKS
   ↓
Rolling Update
   ↓
Smoke Test
```

**Important:** Jenkins is manually triggered in the current implementation. GitHub webhook automation was not implemented.

---

# 5. Technology Stack

| Area | Technology |
|---|---|
| Cloud | AWS |
| Infrastructure as Code | Terraform |
| Containerization | Docker |
| Container Registry | Amazon ECR |
| Kubernetes | Amazon EKS |
| Kubernetes Packaging | Helm |
| Ingress / Load Balancing | AWS Load Balancer Controller + AWS ALB |
| CI/CD | Jenkins |
| Monitoring | Prometheus |
| Visualization / Alerting | Grafana |
| Application | Docker Example Voting App |
| Source Control | Git / GitHub |
| OS / CLI | Linux/Bash + Windows development environment |

---

# 6. Repository Structure

```text
devops-voting-platform/
│
├── docs/
│   ├── developer-architecture.md
│   ├── infastructure-architehture.md
│   ├── runtime-flow.md
│   │
│   ├── phase1/
│   ├── security/
│   ├── troubleshooting/
│   ├── monitoring/
│   ├── ci-cd/
│   ├── interview/
│   └── resume/
│
├── helm/
│   └── voting-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│
├── jenkins/
│   └── Jenkinsfile
│
├── kubernetes/
│   ├── db-deployment.yaml
│   ├── db-pvc.yaml
│   ├── db-secret.yaml
│   ├── db-service.yaml
│   ├── ingress.yaml
│   ├── redis-deployment.yaml
│   ├── redis-service.yaml
│   ├── result-deployment.yaml
│   ├── result-service.yaml
│   ├── vote-deployment.yaml
│   ├── vote-service.yaml
│   └── worker-deployment.yaml
│
├── monitoring/
│
├── scripts/
│
├── terraform/
│   ├── main.tf
│   ├── providers.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── aws-load-balancer-controller-policy.json
│
├── .gitignore
└── README.md
```

---

# 7. Infrastructure with Terraform

Terraform manages the AWS infrastructure required by the platform.

The Terraform configuration includes infrastructure for:

- VPC
- Public and private subnets
- Internet Gateway
- Route tables
- NAT Gateway
- Security Groups
- EKS cluster
- EKS node group
- IAM roles
- EBS CSI integration
- EKS Pod Identity
- AWS Load Balancer Controller IAM configuration
- Helm provider configuration
- AWS Load Balancer Controller Helm release

## Why Terraform?

Manual infrastructure creation becomes difficult to reproduce and audit.

Terraform provides:

```text
Configuration
     ↓
terraform plan
     ↓
Review
     ↓
terraform apply
     ↓
Infrastructure
```

It also allows the environment to be destroyed after testing:

```text
terraform destroy
```

This was especially important for controlling AWS cost during the project.

---

# 8. AWS EKS Architecture

The Kubernetes platform runs on Amazon EKS.

The main components are:

```text
AWS
 |
 +-- VPC
 |
 +-- EKS Control Plane
 |
 +-- EKS Node Group
       |
       +-- Worker Node
       +-- Worker Node
       +-- Worker Node
```

The node group was increased from two to three nodes during monitoring work because Kubernetes Pod capacity was exhausted on the original two-node configuration.

This was a real scheduling problem encountered during the project.

---

# 9. IAM and Security Model

IAM was designed around separation of trust and permissions.

### Trust policy

Defines:

> Who is allowed to assume the role?

### Permission policy

Defines:

> What is the role allowed to do?

Examples from the project:

```text
EKS Cluster Role
    trusted principal → eks.amazonaws.com

EKS Node Role
    trusted principal → ec2.amazonaws.com

Pod Identity Roles
    trusted principal → pods.eks.amazonaws.com
```

The project also uses EKS Pod Identity for workloads such as:

- AWS Load Balancer Controller
- EBS CSI integration

The principle of least privilege was applied conceptually and through the permissions assigned to the platform components.

For example, a runtime node that needs to pull images from ECR does not need permission to push images to ECR.

---

# 10. Container Image Management

The application image is built using Docker and pushed to Amazon ECR.

The flow is:

```text
Application Source
       ↓
Dockerfile
       ↓
docker build
       ↓
Docker Image
       ↓
ECR
       ↓
EKS
```

Jenkins creates build-specific image tags such as:

```text
jenkins-7
```

The Helm deployment then receives the image tag and performs the Kubernetes rollout.

---

# 11. Kubernetes Application Platform

The voting application is deployed as multiple Kubernetes workloads.

The major components include:

- Vote
- Result
- Worker
- Redis
- PostgreSQL

Kubernetes resources used include:

- Deployments
- Services
- Ingress
- PersistentVolumeClaim
- Secret

The Kubernetes layer provides:

- desired state
- replica management
- service discovery
- workload scheduling
- rolling updates
- container restart/recovery behavior
- persistent storage for PostgreSQL

---

# 12. Helm

The application was converted into a Helm chart:

```text
helm/voting-app/
```

Helm provides configurable deployment templates instead of maintaining only static manifests.

Important configurable values include:

- image repository
- image tag
- replica count
- resource requests
- resource limits

The chart also includes:

- health probes
- rolling update configuration
- Kubernetes Secret
- application Services
- database resources
- Ingress

Useful validation commands used during development included:

```bash
helm lint
helm template
helm upgrade --install
```

---

# 13. AWS Load Balancer Controller and Ingress

External traffic enters through an AWS Application Load Balancer.

```text
Internet
   ↓
AWS ALB
   ↓
Kubernetes Ingress
   ↓
Kubernetes Service
   ↓
Pod
```

The AWS Load Balancer Controller watches Kubernetes Ingress resources and provisions/configures the AWS ALB.

The controller was configured with IAM and EKS Pod Identity.

## Real failure encountered

The controller initially failed because it attempted to discover the VPC ID through EC2 instance metadata and timed out.

The configuration was corrected by explicitly providing:

```text
vpcId = aws_vpc.main.id
```

The controller then started successfully.

This became one of the project's infrastructure troubleshooting stories.

---

# 14. Jenkins CI/CD

The Jenkins pipeline is stored at:

```text
jenkins/Jenkinsfile
```

## CI flow

```text
Checkout
   ↓
AWS Authentication Test
   ↓
Get Application Source
   ↓
Validate
   ↓
Docker Build
   ↓
ECR Login
   ↓
Tag
   ↓
Push
```

## CD flow

```text
AWS Authentication
       ↓
Update kubeconfig
       ↓
Helm upgrade/install
       ↓
Set image tag
       ↓
Rollout verification
       ↓
ALB smoke test
```

## Deployment verification

Jenkins waits for the vote Deployment rollout:

```bash
kubectl rollout status deployment/vote
```

It then obtains the Ingress/ALB hostname and performs an HTTP smoke test.

This means the pipeline does more than simply push an image; it verifies that the new version can actually become available through the external entry point.

---

# 15. Jenkins Credential Handling

AWS credentials are stored in Jenkins using the credential ID:

```text
aws-ecr
```

The Jenkinsfile uses credential binding rather than hard-coding credentials.

AWS-dependent stages explicitly inject the credentials.

## Real failure encountered

The first CD attempt failed because credentials were not available outside the original credential scope.

The deployment, rollout verification and smoke-test stages were corrected to explicitly use the credential binding.

This demonstrated an important Jenkins principle:

> Credential availability is scoped. A credential injected into one stage should not be assumed to remain available in another stage.

---

# 16. Monitoring with Prometheus

The project uses the Prometheus monitoring stack.

The flow is:

```text
Kubernetes / Exporters
        ↓
    Prometheus
        ↓
      PromQL
```

Prometheus was used to inspect:

- target health
- target count
- node CPU metrics

Example:

```promql
count(up)
```

The `up` metric was used to understand whether Prometheus targets were being successfully scraped.

Node CPU usage was queried using:

```promql
100 - (
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

---

# 17. Grafana

Grafana was connected to Prometheus as its data source.

The monitoring flow becomes:

```text
Kubernetes
    ↓
Prometheus
    ↓
PromQL
    ↓
Grafana
    ↓
Dashboard
```

Dashboards/panels were created for:

- Node CPU Usage
- Node Memory Usage
- Prometheus Target Health

---

# 18. Alerting

A `High Node CPU Usage` alert was created.

The alert used the node CPU PromQL query and an initial threshold of:

```text
80%
```

For testing, the threshold was temporarily changed to:

```text
1%
```

This caused the alert to enter:

```text
Firing
```

The threshold was restored to 80%, and the alert returned to:

```text
Normal
```

This verified the alert rule lifecycle.

## Notification limitation

The Grafana contact point delivery attempt failed because SMTP was not configured.

This is an important distinction:

```text
Alert rule → worked
Notification delivery → not configured successfully
```

SMTP was intentionally not implemented as part of this project phase.

---

# 19. Real Failure Engineering

One of the most important parts of this project was deliberately breaking the platform and troubleshooting it.

The goal was not just:

> "I know Kubernetes commands."

The goal was:

> "I can identify a failure, collect evidence, determine the root cause, recover the system, and explain what happened."

---

## 19.1 ImagePullBackOff

A deliberately nonexistent image was deployed.

The Pod entered:

```text
ImagePullBackOff
```

Events showed Kubernetes could not pull the requested image.

### Root cause

The image/repository did not exist.

### Troubleshooting approach

```text
Pod unhealthy
   ↓
kubectl describe pod
   ↓
Inspect Events
   ↓
Image pull error
   ↓
Invalid image reference
   ↓
Correct image
   ↓
Pod Running
```

---

# 20. CrashLoopBackOff

A deliberately failing container was created.

The container printed:

```text
Application starting...
Application crashed!
```

and exited with status 1.

The Pod entered:

```text
CrashLoopBackOff
```

and restarted repeatedly.

### Root cause

The main container process exited with a failure code.

### Troubleshooting

```text
Pod status
   ↓
kubectl describe pod
   ↓
Events
   ↓
kubectl logs
   ↓
Application crashed
   ↓
Fix command/application
   ↓
Verify Running
```

The final Pod reached:

```text
1/1 Running
```

---

# 21. Windows / Git Bash Container Command Failure

During failure testing, a shell command was unintentionally transformed by Git Bash on Windows.

Instead of:

```text
/bin/sh
```

the container received a Windows-style path:

```text
C:/Program Files/Git/usr/bin/sh
```

The container failed with:

```text
RunContainerError
```

### Lesson

The problem was not Kubernetes itself.

The command had been transformed before reaching the container.

The command was corrected to:

```text
/bin/sh
```

and the workload recovered.

This was a useful reminder to inspect the exact command received by the container rather than assuming the command in the terminal was identical to the command Kubernetes executed.

---

# 22. Prometheus Pod Capacity Failure

During installation of the monitoring stack, monitoring Pods remained Pending.

The Kubernetes Events showed:

```text
0/2 nodes are available: 2 Too many pods.
```

The nodes had:

```text
Pod capacity: 11
Non-terminated Pods: 11
```

### Root cause

The nodes had exhausted their Pod capacity.

### Fix

The Terraform node group was increased to three nodes.

After the third node joined, the Prometheus/Grafana stack became healthy.

### Lesson

A Kubernetes workload can remain Pending because of Pod capacity even when CPU or memory does not appear to be the immediate problem.

Always inspect scheduler Events.

---

# 23. EKS kubeconfig Failure

After recreating the EKS cluster, `kubectl` initially attempted to contact the old cluster endpoint.

The result was a DNS/API connection failure.

### Root cause

Local kubeconfig referenced the previous cluster.

### Recovery

```bash
aws sts get-caller-identity

aws eks update-kubeconfig \
  --region ap-south-1 \
  --name devops-voting-eks

kubectl get nodes
```

### Lesson

When an EKS cluster is recreated, refresh kubeconfig before assuming the cluster itself is unavailable.

---

# 24. Helm Database Secret Failure

The Helm deployment initially failed because the database Deployment referenced:

```text
db-secret
```

but the Secret was not part of the Helm release.

The Secret existed as a raw Kubernetes manifest but was missing from the Helm chart.

### Fix

The Secret was added to:

```text
helm/voting-app/templates/db-secret.yaml
```

Then the chart was validated with:

```bash
helm lint
helm template
```

and deployed successfully.

### Lesson

Resources required by a Helm release should be included in the release's deployment contract rather than relying on an unrelated manually applied manifest.

---

# 25. AWS Cleanup Failure

AWS resources were intentionally destroyed after learning sessions to control cost.

During one cleanup, Terraform initially left an Application Load Balancer and Kubernetes-created network resources.

The issue demonstrated an important dependency:

```text
Kubernetes Ingress
        ↓
AWS ALB
        ↓
AWS networking resources
```

The ALB had to be removed before the remaining network resources and VPC could be fully cleaned up.

### Final cleanup principle

```text
Uninstall application
        ↓
Remove Ingress
        ↓
Wait for ALB deletion
        ↓
Verify ENIs / target groups / SGs
        ↓
Terraform destroy
        ↓
Verify AWS
        ↓
Verify Terraform state
```

This became an important operational lesson:

> Never destroy the underlying EKS infrastructure before cleaning up Kubernetes resources that provision AWS resources.

---

# 26. Standard Troubleshooting Method

The project follows an evidence-first troubleshooting approach.

```text
1. Identify the unhealthy resource
             ↓
2. Describe it
             ↓
3. Inspect Events
             ↓
4. Check logs
             ↓
5. Identify root cause
             ↓
6. Apply targeted fix
             ↓
7. Verify recovery
             ↓
8. Document the incident
```

Examples:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events --sort-by=.lastTimestamp
```

The principle is:

> **Do not randomly change commands. Inspect evidence first.**

---

# 27. Security Practices and Current Gaps

## Implemented / demonstrated

- IAM role-based access
- IAM trust vs permission separation
- Least-privilege concepts
- EKS Pod Identity
- Jenkins credential binding
- Kubernetes Secret for database credentials
- AWS IAM instead of storing AWS credentials directly on EKS nodes
- Private infrastructure components where applicable

## Discussed but not implemented

The following are documented future improvements rather than completed features:

- Container image vulnerability scanning
- AWS Secrets Manager integration
- Complete Kubernetes RBAC implementation
- SMTP notification delivery
- Automated GitHub webhook triggering

The project intentionally documents these limitations instead of claiming them as implemented.

---

# 28. Reliability Features

The application deployment includes reliability-oriented configuration such as:

- multiple vote replicas
- resource requests/limits
- health probes
- rolling updates
- Kubernetes desired-state management
- persistent storage for PostgreSQL
- automated container restart behavior
- rollout verification in Jenkins

The project also exercised failure recovery rather than only normal operation.

---

# 29. Deployment Verification

A successful deployment is not considered complete merely because `helm upgrade` returns successfully.

The Jenkins pipeline verifies:

### Kubernetes rollout

```bash
kubectl rollout status deployment/vote
```

### Pod health

```bash
kubectl get pods -n voting
```

### External application path

The pipeline retrieves the ALB hostname and performs an HTTP smoke test.

Therefore the deployment verification path is:

```text
Helm deployment
      ↓
Kubernetes rollout
      ↓
Pods healthy
      ↓
Ingress/ALB available
      ↓
HTTP smoke test
```

---

# 30. AWS Cost-Control Discipline

Because AWS resources can incur charges, the project follows a cleanup discipline after learning sessions.

The expected sequence is:

```text
Remove monitoring
      ↓
Remove application
      ↓
Remove Ingress / ALB
      ↓
Verify AWS networking resources
      ↓
terraform destroy
      ↓
Verify EKS
      ↓
Verify NAT / VPC / ENIs
      ↓
Verify Terraform state
```

The environment was destroyed after the relevant exercises rather than being left running unnecessarily.

---

# 31. Project Limitations

This is a production-style learning and interview project, not a production system serving real customers.

The following are known limitations:

1. Jenkins is manually triggered.
2. GitHub webhook automation is not implemented.
3. Prometheus/Grafana were installed manually with Helm rather than fully managed by Terraform.
4. Grafana SMTP notification delivery was not configured.
5. Image vulnerability scanning was discussed but not implemented.
6. AWS Secrets Manager integration was discussed but not implemented.
7. Full Kubernetes RBAC hardening was not implemented.
8. The application source is an external Example Voting App; the project focuses on its DevOps platform.
9. GitOps/ArgoCD was not implemented.

These limitations are intentional and documented so that the project remains technically honest.

---

# 32. What I Learned From the Project

This project moved beyond individual tool commands and focused on the relationships between DevOps components.

### Infrastructure

I learned how Terraform can define and recreate AWS infrastructure and how AWS services fit together to provide a Kubernetes platform.

### Kubernetes

I learned how Deployments, Pods, Services, Ingress, Secrets and persistent storage work together to run an application.

### CI/CD

I learned how Jenkins can connect source retrieval, Docker image creation, ECR publishing, Helm deployment and post-deployment verification into one pipeline.

### Monitoring

I learned how Prometheus collects metrics and how Grafana queries Prometheus through PromQL to visualize and alert on operational data.

### Troubleshooting

Most importantly, I learned to troubleshoot from evidence:

```text
Symptom
  ↓
Evidence
  ↓
Root Cause
  ↓
Targeted Fix
  ↓
Verification
```

# 33. Future Improvements

Potential next-stage improvements include:

- GitHub webhook-triggered Jenkins builds
- container image vulnerability scanning
- external secret management with AWS Secrets Manager
- stronger Kubernetes RBAC
- automated monitoring provisioning
- Grafana SMTP/notification integration
- centralized logging
- additional SLO/SLI monitoring
- infrastructure and application deployment separation
- GitOps with Argo CD

These are future improvements and are not represented as completed features in the current project.


# 36. Final Project Definition

This project demonstrates an end-to-end DevOps workflow:

```text
Source
  ↓
CI
  ↓
Docker
  ↓
ECR
  ↓
CD
  ↓
Helm
  ↓
Kubernetes / EKS
  ↓
AWS ALB
  ↓
Application
  ↓
Prometheus
  ↓
Grafana
  ↓
Alerting
```

Surrounding the workflow are:

```text
Terraform
IAM
Security
Reliability
Troubleshooting
Failure Engineering
Operational Cleanup
```

The project is designed to demonstrate not only deployment knowledge, but also the operational mindset required to maintain and troubleshoot a cloud-native platform.
