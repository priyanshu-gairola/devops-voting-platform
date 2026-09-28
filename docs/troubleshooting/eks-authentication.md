# EKS Authentication / kubeconfig Incident

## Symptom
After recreating the EKS cluster, kubectl attempted to contact a stale/nonexistent EKS endpoint and failed DNS resolution.

## Root cause
The local kubeconfig still referenced the previous cluster endpoint.

## Recovery
Refresh kubeconfig:

```bash
aws sts get-caller-identity
aws eks update-kubeconfig --region ap-south-1 --name devops-voting-eks
kubectl get nodes -o wide
```

## Lesson
After recreating an EKS cluster, refresh kubeconfig before assuming the cluster itself is unhealthy.
