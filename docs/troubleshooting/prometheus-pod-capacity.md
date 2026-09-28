# Prometheus Installation / Pod Capacity Incident

## Symptom
The kube-prometheus-stack installation failed because an admission Job could not schedule.

## Evidence
Events reported:
`0/2 nodes are available: 2 Too many pods.`

The nodes each had pod capacity 11 and were already full.

## Root cause
Node pod capacity was exhausted.

## Recovery
Terraform changed the node group desired/min/max configuration to support three nodes.

After the third node joined, the monitoring stack became healthy.

## Lesson
Pod scheduling failures can be caused by node capacity even when CPU and memory are not obviously exhausted. Check node Pod capacity and Events.
