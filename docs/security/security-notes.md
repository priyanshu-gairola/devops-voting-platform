# Security Notes

## IAM
- Trust policy: who can assume a role.
- Permission policy: what the role can do.
- Least privilege: grant only required permissions.

## Project examples
EKS nodes require ECR pull capability; they do not need ECR push capability for the runtime workload.

AWS Load Balancer Controller and EBS CSI use EKS Pod Identity.

## Secrets
Database credentials are represented as a Kubernetes Secret in the project.

AWS Secrets Manager was discussed as a stronger external secret-management approach but is not integrated.

## Container security
Image scanning and non-root execution were discussed as improvements.

**Not implemented:** image scanning.

## Security claims
 The project demonstrates security concepts and selected controls rather than a complete enterprise security program.
