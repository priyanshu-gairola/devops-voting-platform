# Helm Database Secret Incident

## Symptom
The Helm deployment failed because the database Deployment referenced a Secret that did not exist in the target namespace.

## Root cause
The Secret existed as a raw Kubernetes manifest but had not been included in the Helm chart.

## Recovery
The Secret definition was moved into:
`helm/voting-app/templates/db-secret.yaml`

Then:
- `helm lint`
- `helm template`
- `helm upgrade`

were used to validate and deploy the correction.

## Lesson
Helm should own the resources required by the release when those resources are part of the application's deployment contract.
