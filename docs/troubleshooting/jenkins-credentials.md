# Jenkins Credential Scope Incident

## Symptom
Jenkins CI succeeded, but CD stages failed when AWS credentials were unavailable.

## Root cause
The AWS credential binding was scoped only around selected stages.

## Recovery
AWS credentials were explicitly injected with `withCredentials` in the required AWS-dependent stages.

Affected stages included:
- Deploy to EKS
- Verify Rollout
- Smoke Test

## Lesson
Credential availability is stage-scoped. Never assume credentials injected in one Jenkins stage remain available in another.
