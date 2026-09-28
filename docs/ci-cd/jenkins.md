# Jenkins CI/CD

## Pipeline

```text
Checkout
   ↓
Validate
   ↓
Docker Build
   ↓
ECR Login
   ↓
Tag + Push
   ↓
EKS kubeconfig
   ↓
Helm Deploy
   ↓
Rollout Verification
   ↓
ALB Smoke Test
```

## CI
Jenkins obtains the external Example Voting App source, validates the vote service files, builds the image, authenticates to ECR, and pushes a build-specific image tag.

## CD
Jenkins updates kubeconfig, deploys the image through Helm, waits for rollout completion, and performs an HTTP smoke test through the ALB.

## Credentials
AWS credentials are stored in Jenkins as `aws-ecr` and injected into required stages.

## Limitation
The pipeline is manually triggered. GitHub webhook automation was not implemented.
