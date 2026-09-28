# AWS Cleanup Incident

## Context
AWS resources were intentionally destroyed after learning exercises to avoid unnecessary charges.

## Incident
Terraform destroy initially left an Application Load Balancer and Kubernetes-created ENIs/Security Groups.

The ALB was manually deleted after the EKS cluster was already gone. The remaining Kubernetes-created Security Groups were found to be unreferenced and removed. The VPC could then be deleted.

Terraform state was reconciled afterward.

## Critical lesson
Destroy Kubernetes applications and Ingress/ALB before destroying the EKS infrastructure.

## Required cleanup sequence
1. Uninstall monitoring.
2. Uninstall voting application.
3. Wait for Ingress/ALB deletion.
4. Verify ALB/ENIs/target groups.
5. Run Terraform destroy.
6. Verify AWS resources.
7. Verify Terraform state is empty.
8. Verify Git working tree is clean.
