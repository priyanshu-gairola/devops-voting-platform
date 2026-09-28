# Incident Troubleshooting Runbook

Use evidence before changing configuration.

## Kubernetes workload
1. `kubectl get pods`
2. Identify unhealthy Pod.
3. `kubectl describe pod <pod>`
4. Inspect Events.
5. `kubectl logs <pod>`
6. Check Deployment/ReplicaSet if needed.
7. Fix the root cause.
8. Verify rollout and application health.

## Common symptoms
- `Pending` → scheduling/resource/capacity issue.
- `ImagePullBackOff` → image pull problem.
- `CrashLoopBackOff` → container repeatedly exits.
- `RunContainerError` → container failed before normal execution.
- unhealthy Ingress/ALB → inspect Ingress, controller logs, target health, and AWS resources.

## Project principle
Do not randomly change commands. Gather evidence first.
