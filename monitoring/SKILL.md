---
name: monitoring
description: "Set up monitoring, logging, and observability for applications. Use when implementing dashboards, alerting, metrics collection, or troubleshooting."
---

# Monitoring Skill

Implement monitoring, logging, and observability for applications.

## When to Use

Use this skill when the user wants to:
- Set up metrics collection
- Create monitoring dashboards
- Configure alerting rules
- Implement structured logging
- Set up distributed tracing
- Work with observability tools
- Performance monitoring

## Monitoring Stack Options

### Agent-based
- **Prometheus**: Metrics collection
- **Grafana**: Visualization and dashboards
- **Node Exporter**: Node metrics
- **CAdvisor**: Container metrics

### Serverless
- **CloudWatch**: AWS monitoring
- **Cloud Logging**: Google Cloud Logging
- **App Insights**: Azure Application Insights

### Distributed Tracing
- **Jaeger**: OpenTracing compatible
- **Zipkin**: Distributed tracing
- **OpenTelemetry**: Tracing standard

## Metrics Types

### Metrics
- Counter: Monotonically increasing
- Gauge: Can go up or down
- Histogram: Value distribution
- Summary: Value distribution + quantiles

### Logging
- Structured logs (JSON)
- Log levels (ERROR, WARN, INFO, DEBUG)
- Correlation IDs for tracing
- Log rotation and retention

## Best Practices

- **Log everything important**: Errors, warnings, user actions
- **Use meaningful log levels**: Don't spam with DEBUG
- **Structure logs**: JSON format with fields
- **Don't log secrets**: Never log passwords, tokens, API keys
- **Add context**: Include request ID, user info, timestamps
- **Monitor what matters**: Core business metrics, errors, SLIs

## Alerting Strategy

- **Alarm on anomalies**: Systematic errors, unusual patterns
- **Set reasonable thresholds**: Avoid alert fatigue
- **Escalate appropriately**: Critical issues get attention
- **Keep SLOs in mind**: Alerts should align with SLOs

## Deliverables

- Monitoring configuration files
- Dashboard definition
- Alerting rules
- Log aggregation setup
- Documentation

## Quality Checklist

- Critical metrics are monitored
- Alerts are actionable
- Logs are structured and searchable
- Metrics are properly labeled
- Distributed tracing is configured
- SLO/SLA alignment
