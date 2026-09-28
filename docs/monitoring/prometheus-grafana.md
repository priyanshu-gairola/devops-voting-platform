# Monitoring — Prometheus and Grafana

## Architecture

```text
Kubernetes / Exporters
        ↓
    Prometheus
        ↓
      PromQL
        ↓
     Grafana
        ↓
 Dashboard / Alert
```

## Prometheus
Prometheus collects and stores time-series metrics and provides PromQL queries.

Examples:
- `up`
- `count(up)`
- node CPU query

## Grafana
Grafana queries Prometheus and visualizes the results in dashboards and panels.

## Dashboards created
- Node CPU Usage
- Node Memory Usage
- Prometheus Target Health

## Alert
High Node CPU Usage was tested by temporarily lowering the threshold to 1%. The alert entered Firing and returned to Normal after the threshold was restored.

## Notification limitation
SMTP was not configured, so email delivery was not completed. The alert rule itself functioned.

## Deployment limitation
The monitoring stack was installed manually with Helm and was not Terraform-managed.
