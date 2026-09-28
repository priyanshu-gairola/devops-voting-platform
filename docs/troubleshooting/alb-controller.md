# AWS Load Balancer Controller Incident

## Symptom
The AWS Load Balancer Controller failed during initialization while trying to discover the VPC ID through EC2 instance metadata.

## Error
The controller reported a timeout while fetching VPC metadata.

## Root cause
VPC discovery through metadata was unreliable during initialization.

## Recovery
Terraform was changed to pass the VPC explicitly:

```text
vpcId = aws_vpc.main.id
```

The controller then started successfully.

## Lesson
For critical infrastructure controllers, explicit infrastructure identifiers can avoid runtime discovery dependencies.
