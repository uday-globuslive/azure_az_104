# Module 22: Platform Reliability and Observability

## Table of Contents
1. [SRE Principles and SLOs](#sre-principles)
2. [Observability Architecture](#observability-architecture)
3. [Prometheus and Grafana](#prometheus-grafana)
4. [Azure Monitor and Log Analytics](#azure-monitor)
5. [Distributed Tracing](#distributed-tracing)
6. [Incident Management and MTTR](#incident-management)
7. [Chaos Engineering](#chaos-engineering)
8. [Interview Scenarios](#interview-scenarios)

---

## 1. SRE Principles and SLOs <a name="sre-principles"></a>

### SRE Framework
```
┌─────────────────────────────────────────────────────────────────┐
│                    SRE Principles                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SLI (Service Level Indicator)                                  │
│  ├── Availability: % of successful requests                     │
│  ├── Latency: Response time at various percentiles             │
│  ├── Throughput: Requests per second                           │
│  └── Error Rate: % of failed requests                          │
│                                                                  │
│  SLO (Service Level Objective)                                  │
│  ├── Target value for an SLI                                   │
│  ├── Example: 99.9% availability over 30 days                  │
│  └── Defines acceptable performance                            │
│                                                                  │
│  SLA (Service Level Agreement)                                  │
│  ├── Contractual commitment                                    │
│  ├── Usually less strict than SLO                              │
│  └── Financial penalties for breach                            │
│                                                                  │
│  Error Budget                                                   │
│  ├── Allowed failures: 100% - SLO                              │
│  ├── 99.9% SLO = 43.2 minutes downtime/month                   │
│  └── Balances reliability vs. feature velocity                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### SLO Definition Examples
```yaml
# slo-definitions.yaml
service: web-application
team: platform-engineering

slos:
  - name: availability
    description: "Successful HTTP responses"
    indicator:
      type: ratio
      good_events: |
        sum(rate(http_requests_total{status=~"2..|3.."}[5m]))
      total_events: |
        sum(rate(http_requests_total[5m]))
    objective: 99.9
    window: 30d
    
  - name: latency-p99
    description: "99th percentile response time"
    indicator:
      type: threshold
      metric: |
        histogram_quantile(0.99, 
          sum(rate(http_request_duration_seconds_bucket[5m])) by (le))
    objective:
      target: 0.5  # 500ms
      threshold_type: less_than
    window: 30d
    
  - name: error-rate
    description: "Server error rate (5xx)"
    indicator:
      type: ratio
      bad_events: |
        sum(rate(http_requests_total{status=~"5.."}[5m]))
      total_events: |
        sum(rate(http_requests_total[5m]))
    objective: 0.1  # Less than 0.1% errors
    window: 7d

error_budget:
  - name: monthly-budget
    slo_ref: availability
    calculation: |
      1 - (slo_objective / 100)  # 0.1% = 43.2 minutes
    alerts:
      - threshold: 50  # 50% consumed
        severity: warning
      - threshold: 80  # 80% consumed
        severity: critical
        action: freeze_deployments
```

### Error Budget Calculation
```python
# error_budget_calculator.py
from datetime import datetime, timedelta
from dataclasses import dataclass
from typing import Dict, List
import prometheus_client

@dataclass
class SLOConfig:
    name: str
    objective: float  # e.g., 99.9
    window_days: int
    good_events_query: str
    total_events_query: str

class ErrorBudgetCalculator:
    def __init__(self, prometheus_url: str):
        self.prom = prometheus_client.PrometheusClient(prometheus_url)
        
    def calculate_error_budget(self, slo: SLOConfig) -> Dict:
        """Calculate error budget status"""
        
        window_start = datetime.now() - timedelta(days=slo.window_days)
        
        # Query Prometheus for events
        good_events = self.prom.query(slo.good_events_query, window_start)
        total_events = self.prom.query(slo.total_events_query, window_start)
        
        # Calculate actual performance
        actual_availability = (good_events / total_events) * 100 if total_events > 0 else 100
        
        # Error budget calculation
        allowed_failures = 100 - slo.objective  # e.g., 0.1% for 99.9% SLO
        actual_failures = 100 - actual_availability
        
        budget_remaining = ((allowed_failures - actual_failures) / allowed_failures) * 100
        
        # Time calculations
        window_minutes = slo.window_days * 24 * 60
        allowed_downtime_minutes = (allowed_failures / 100) * window_minutes
        consumed_downtime_minutes = (actual_failures / 100) * window_minutes
        remaining_downtime_minutes = allowed_downtime_minutes - consumed_downtime_minutes
        
        return {
            "slo_name": slo.name,
            "objective": slo.objective,
            "actual_availability": round(actual_availability, 4),
            "error_budget_remaining_percent": round(budget_remaining, 2),
            "allowed_downtime_minutes": round(allowed_downtime_minutes, 2),
            "consumed_downtime_minutes": round(consumed_downtime_minutes, 2),
            "remaining_downtime_minutes": round(remaining_downtime_minutes, 2),
            "status": self._get_status(budget_remaining)
        }
    
    def _get_status(self, budget_remaining: float) -> str:
        if budget_remaining > 50:
            return "healthy"
        elif budget_remaining > 20:
            return "warning"
        elif budget_remaining > 0:
            return "critical"
        else:
            return "exhausted"
```

---

## 2. Observability Architecture <a name="observability-architecture"></a>

### Three Pillars of Observability
```
┌─────────────────────────────────────────────────────────────────┐
│                  Observability Stack                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │   Metrics   │    │    Logs     │    │   Traces    │          │
│  │  (Numbers)  │    │   (Text)    │    │  (Context)  │          │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘          │
│         │                  │                  │                  │
│         ▼                  ▼                  ▼                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │ Prometheus  │    │   Loki/     │    │   Jaeger/   │          │
│  │   Thanos    │    │   ELK       │    │   Tempo     │          │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘          │
│         │                  │                  │                  │
│         └─────────────────┬┴──────────────────┘                  │
│                           │                                      │
│                    ┌──────┴──────┐                               │
│                    │   Grafana   │                               │
│                    │ (Unified UI)│                               │
│                    └─────────────┘                               │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  Alerting: Prometheus Alertmanager / Grafana / PagerDuty        │
└─────────────────────────────────────────────────────────────────┘
```

### OpenTelemetry Integration
```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
  
  prometheus:
    config:
      scrape_configs:
        - job_name: 'kubernetes-pods'
          kubernetes_sd_configs:
            - role: pod
          relabel_configs:
            - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
              action: keep
              regex: true

  filelog:
    include: [/var/log/containers/*.log]
    start_at: beginning
    operators:
      - type: json_parser
        timestamp:
          parse_from: attributes.time
          layout: '%Y-%m-%dT%H:%M:%S.%fZ'

processors:
  batch:
    timeout: 10s
    send_batch_size: 1000
  
  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200
  
  resourcedetection:
    detectors: [env, system, docker, azure]
  
  attributes:
    actions:
      - key: environment
        value: production
        action: insert

exporters:
  prometheus:
    endpoint: "0.0.0.0:8889"
  
  loki:
    endpoint: http://loki:3100/loki/api/v1/push
  
  jaeger:
    endpoint: jaeger-collector:14250
    tls:
      insecure: true
  
  azuremonitor:
    connection_string: ${APPLICATIONINSIGHTS_CONNECTION_STRING}

service:
  pipelines:
    metrics:
      receivers: [otlp, prometheus]
      processors: [memory_limiter, batch, resourcedetection]
      exporters: [prometheus, azuremonitor]
    
    logs:
      receivers: [otlp, filelog]
      processors: [memory_limiter, batch, attributes]
      exporters: [loki, azuremonitor]
    
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [jaeger, azuremonitor]
```

---

## 3. Prometheus and Grafana <a name="prometheus-grafana"></a>

### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: production-east
    environment: production

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - '/etc/prometheus/rules/*.yml'

scrape_configs:
  - job_name: 'kubernetes-apiservers'
    kubernetes_sd_configs:
      - role: endpoints
    scheme: https
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
```

### Prometheus Alert Rules
```yaml
# alerts/application-alerts.yml
groups:
  - name: application-slo
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) > 0.01
        for: 5m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} (threshold: 1%)"
          runbook_url: "https://wiki.corp.local/runbooks/high-error-rate"
      
      - alert: HighLatencyP99
        expr: |
          histogram_quantile(0.99, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
          ) > 1
        for: 5m
        labels:
          severity: warning
          team: platform
        annotations:
          summary: "High p99 latency for {{ $labels.service }}"
          description: "P99 latency is {{ $value | humanizeDuration }}"
      
      - alert: ErrorBudgetBurn
        expr: |
          (
            1 - (
              sum(rate(http_requests_total{status=~"2..|3.."}[1h]))
              /
              sum(rate(http_requests_total[1h]))
            )
          ) > (1 - 0.999) * 14.4
        for: 5m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "Error budget burning too fast"
          description: "At current rate, error budget will be exhausted in less than 7 days"

  - name: infrastructure
    rules:
      - alert: NodeHighCPU
        expr: |
          100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          
      - alert: NodeHighMemory
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          
      - alert: DiskSpaceLow
        expr: |
          (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 10
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
```

### Grafana Dashboard Configuration
```json
{
  "dashboard": {
    "title": "Platform SLO Dashboard",
    "panels": [
      {
        "title": "Error Budget Status",
        "type": "gauge",
        "gridPos": {"x": 0, "y": 0, "w": 6, "h": 8},
        "targets": [
          {
            "expr": "100 * (1 - ((1 - sum(rate(http_requests_total{status=~\"2..|3..\"}[30d])) / sum(rate(http_requests_total[30d]))) / 0.001))",
            "legendFormat": "Error Budget Remaining"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {"color": "red", "value": 0},
                {"color": "orange", "value": 20},
                {"color": "yellow", "value": 50},
                {"color": "green", "value": 75}
              ]
            },
            "unit": "percent",
            "max": 100,
            "min": 0
          }
        }
      },
      {
        "title": "Request Rate",
        "type": "graph",
        "gridPos": {"x": 6, "y": 0, "w": 9, "h": 8},
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (status_code)",
            "legendFormat": "{{status_code}}"
          }
        ]
      },
      {
        "title": "Latency Distribution",
        "type": "heatmap",
        "gridPos": {"x": 15, "y": 0, "w": 9, "h": 8},
        "targets": [
          {
            "expr": "sum(rate(http_request_duration_seconds_bucket[5m])) by (le)"
          }
        ]
      }
    ]
  }
}
```

---

## 4. Azure Monitor and Log Analytics <a name="azure-monitor"></a>

### Azure Monitor Configuration
```hcl
# Log Analytics Workspace
resource "azurerm_log_analytics_workspace" "main" {
  name                = "platform-logs"
  location            = azurerm_resource_group.monitoring.location
  resource_group_name = azurerm_resource_group.monitoring.name
  sku                 = "PerGB2018"
  retention_in_days   = 90
  
  daily_quota_gb = 10
}

# Application Insights
resource "azurerm_application_insights" "main" {
  name                = "platform-appinsights"
  location            = azurerm_resource_group.monitoring.location
  resource_group_name = azurerm_resource_group.monitoring.name
  workspace_id        = azurerm_log_analytics_workspace.main.id
  application_type    = "web"
}

# Action Group for Alerts
resource "azurerm_monitor_action_group" "critical" {
  name                = "critical-alerts"
  resource_group_name = azurerm_resource_group.monitoring.name
  short_name          = "critical"
  
  email_receiver {
    name                    = "platform-team"
    email_address           = "platform-team@corp.local"
    use_common_alert_schema = true
  }
  
  webhook_receiver {
    name        = "pagerduty"
    service_uri = "https://events.pagerduty.com/integration/xxx/enqueue"
  }
  
  azure_app_push_receiver {
    name          = "mobile-push"
    email_address = "oncall@corp.local"
  }
}

# Metric Alert
resource "azurerm_monitor_metric_alert" "high_cpu" {
  name                = "high-cpu-alert"
  resource_group_name = azurerm_resource_group.monitoring.name
  scopes              = [azurerm_kubernetes_cluster.main.id]
  description         = "Alert when CPU usage exceeds 85%"
  severity            = 2
  frequency           = "PT5M"
  window_size         = "PT15M"
  
  criteria {
    metric_namespace = "Insights.Container/nodes"
    metric_name      = "cpuUsagePercentage"
    aggregation      = "Average"
    operator         = "GreaterThan"
    threshold        = 85
  }
  
  action {
    action_group_id = azurerm_monitor_action_group.critical.id
  }
}

# Log Alert
resource "azurerm_monitor_scheduled_query_rules_alert_v2" "error_spike" {
  name                = "error-spike-alert"
  location            = azurerm_resource_group.monitoring.location
  resource_group_name = azurerm_resource_group.monitoring.name
  scopes              = [azurerm_log_analytics_workspace.main.id]
  
  evaluation_frequency = "PT5M"
  window_duration      = "PT15M"
  severity             = 1
  
  criteria {
    query                   = <<-QUERY
      AppRequests
      | where ResultCode startswith "5"
      | summarize ErrorCount = count() by bin(TimeGenerated, 5m)
      | where ErrorCount > 100
    QUERY
    time_aggregation_method = "Count"
    threshold               = 0
    operator                = "GreaterThan"
    
    failing_periods {
      minimum_failing_periods_to_trigger_alert = 1
      number_of_evaluation_periods             = 1
    }
  }
  
  action {
    action_groups = [azurerm_monitor_action_group.critical.id]
  }
}
```

### KQL Queries for Log Analytics
```kql
// Application Performance
AppRequests
| where TimeGenerated > ago(1h)
| summarize 
    RequestCount = count(),
    AvgDuration = avg(DurationMs),
    P95Duration = percentile(DurationMs, 95),
    P99Duration = percentile(DurationMs, 99),
    FailureRate = 100.0 * countif(Success == false) / count()
  by bin(TimeGenerated, 5m)
| order by TimeGenerated desc

// Error Analysis
AppExceptions
| where TimeGenerated > ago(24h)
| summarize ExceptionCount = count() by 
    ExceptionType = tostring(split(Type, ",")[0]),
    CloudRoleName
| order by ExceptionCount desc
| take 20

// Dependency Performance
AppDependencies
| where TimeGenerated > ago(1h)
| summarize 
    CallCount = count(),
    AvgDuration = avg(DurationMs),
    FailureRate = 100.0 * countif(Success == false) / count()
  by DependencyType, Target, Name
| order by FailureRate desc

// Container Insights - Node Performance
Perf
| where ObjectName == "K8SNode"
| where CounterName == "cpuUsageNanoCores" or CounterName == "memoryRssBytes"
| summarize AvgValue = avg(CounterValue) by 
    Computer, 
    CounterName, 
    bin(TimeGenerated, 5m)
| render timechart

// Custom SLO Tracking
let slo_target = 99.9;
AppRequests
| where TimeGenerated > ago(30d)
| summarize 
    TotalRequests = count(),
    SuccessfulRequests = countif(ResultCode startswith "2" or ResultCode startswith "3")
| extend 
    Availability = 100.0 * SuccessfulRequests / TotalRequests,
    SLOTarget = slo_target,
    ErrorBudget = 100 - slo_target,
    ErrorBudgetUsed = 100 - Availability,
    ErrorBudgetRemaining = (100 - slo_target) - (100 - Availability)
```

---

## 5. Distributed Tracing <a name="distributed-tracing"></a>

### Jaeger Configuration
```yaml
# jaeger-deployment.yaml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: production-jaeger
  namespace: observability
spec:
  strategy: production
  
  collector:
    replicas: 3
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 1000m
        memory: 1Gi
    options:
      collector:
        num-workers: 100
        queue-size: 10000
  
  query:
    replicas: 2
    resources:
      requests:
        cpu: 200m
        memory: 256Mi
  
  storage:
    type: elasticsearch
    elasticsearch:
      nodeCount: 3
      resources:
        requests:
          cpu: 1
          memory: 4Gi
      storage:
        size: 100Gi
        storageClassName: premium-ssd
    esIndexCleaner:
      enabled: true
      numberOfDays: 14
      schedule: "55 23 * * *"
  
  ingress:
    enabled: true
    hosts:
      - jaeger.corp.local
    tls:
      - secretName: jaeger-tls
        hosts:
          - jaeger.corp.local
```

### Application Tracing Implementation
```python
# Python application with OpenTelemetry tracing
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor
from flask import Flask, request
import requests

# Configure tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

jaeger_exporter = JaegerExporter(
    agent_host_name="jaeger-agent.observability.svc.cluster.local",
    agent_port=6831,
)

trace.get_tracer_provider().add_span_processor(
    BatchSpanProcessor(jaeger_exporter)
)

# Auto-instrument libraries
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
SQLAlchemyInstrumentor().instrument()

@app.route('/api/orders/<order_id>')
def get_order(order_id):
    with tracer.start_as_current_span("get_order") as span:
        span.set_attribute("order.id", order_id)
        
        # Fetch order from database
        with tracer.start_as_current_span("database_query"):
            order = db.query(Order).filter_by(id=order_id).first()
        
        # Call external service
        with tracer.start_as_current_span("inventory_check"):
            inventory = requests.get(
                f"http://inventory-service/api/check/{order.product_id}"
            )
        
        span.set_attribute("order.status", order.status)
        span.set_attribute("inventory.available", inventory.json()['available'])
        
        return {"order": order.to_dict(), "inventory": inventory.json()}
```

---

## 6. Incident Management and MTTR <a name="incident-management"></a>

### Incident Response Automation
```yaml
# incident-response-runbook.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: incident-runbooks
  namespace: observability
data:
  high-error-rate.yaml: |
    name: High Error Rate Response
    severity: P1
    slo_impact: true
    
    investigation:
      - step: Check error distribution
        query: |
          sum by (status_code, path) (
            increase(http_requests_total{status=~"5.."}[15m])
          )
        expected: Identify concentrated errors
        
      - step: Check recent deployments
        action: kubectl rollout history deployment -n production
        
      - step: Check resource saturation
        query: |
          (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m]))) * 100
        threshold: 85
        
      - step: Check dependency health
        query: |
          avg by (target) (
            rate(http_client_requests_total{status=~"5.."}[5m])
          )
    
    mitigation:
      - name: Scale up pods
        condition: error_rate > 5% AND cpu_usage > 80%
        action: |
          kubectl scale deployment web-app --replicas=10 -n production
          
      - name: Rollback deployment
        condition: error_rate > 10% AND recent_deployment
        action: |
          kubectl rollout undo deployment/web-app -n production
          
      - name: Enable circuit breaker
        condition: dependency_error_rate > 50%
        action: |
          kubectl patch configmap feature-flags -p '{"data":{"circuit_breaker_enabled":"true"}}'
    
    communication:
      - channel: "#incidents"
        template: |
          🚨 *P1 Incident: High Error Rate*
          Service: {{ .service }}
          Error Rate: {{ .error_rate }}%
          Start Time: {{ .start_time }}
          On-call: {{ .oncall }}
          
      - escalation:
          - after: 15m
            notify: engineering-leads
          - after: 30m
            notify: vp-engineering
```

### MTTR Dashboard Metrics
```promql
# Mean Time to Detect (MTTD)
# Time from incident start to first alert

# Mean Time to Acknowledge (MTTA)
# Time from alert to acknowledgment

# Mean Time to Resolve (MTTR)
# Time from incident start to resolution

# Prometheus recording rules for MTTR metrics
groups:
  - name: incident-metrics
    rules:
      - record: incident:mttr:minutes
        expr: |
          avg(
            incident_resolved_timestamp_seconds - incident_started_timestamp_seconds
          ) / 60
          
      - record: incident:mttd:minutes
        expr: |
          avg(
            incident_detected_timestamp_seconds - incident_started_timestamp_seconds
          ) / 60
          
      - record: incident:mtta:minutes
        expr: |
          avg(
            incident_acknowledged_timestamp_seconds - incident_detected_timestamp_seconds
          ) / 60
```

---

## 7. Chaos Engineering <a name="chaos-engineering"></a>

### Chaos Mesh Configuration
```yaml
# chaos-experiments/network-delay.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay-test
  namespace: chaos-testing
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - production
    labelSelectors:
      "app": "web-frontend"
  delay:
    latency: "100ms"
    jitter: "20ms"
    correlation: "50"
  duration: "10m"
  scheduler:
    cron: "@every 24h"

---
# chaos-experiments/pod-failure.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure-test
  namespace: chaos-testing
spec:
  action: pod-kill
  mode: one
  selector:
    namespaces:
      - production
    labelSelectors:
      "app": "api-service"
  scheduler:
    cron: "0 10 * * 1-5"  # Weekdays at 10 AM

---
# chaos-experiments/stress-test.yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: cpu-stress-test
  namespace: chaos-testing
spec:
  mode: all
  selector:
    namespaces:
      - production
    labelSelectors:
      "tier": "backend"
  stressors:
    cpu:
      workers: 4
      load: 80
    memory:
      workers: 4
      size: "500MB"
  duration: "5m"
```

### Litmus Chaos Workflow
```yaml
apiVersion: litmuschaos.io/v1alpha1
kind: ChaosEngine
metadata:
  name: resilience-test
  namespace: production
spec:
  appinfo:
    appns: production
    applabel: "app=web-frontend"
    appkind: deployment
  
  chaosServiceAccount: litmus-admin
  
  experiments:
    - name: pod-delete
      spec:
        components:
          env:
            - name: TOTAL_CHAOS_DURATION
              value: "30"
            - name: CHAOS_INTERVAL
              value: "10"
            - name: FORCE
              value: "false"
        probe:
          - name: "check-frontend-availability"
            type: "httpProbe"
            mode: "Continuous"
            httpProbe/inputs:
              url: "http://web-frontend.production.svc.cluster.local/health"
              insecureSkipVerify: false
              method:
                get:
                  criteria: "=="
                  responseCode: "200"
            runProperties:
              probeTimeout: 5
              interval: 5
              retry: 3
```

---

## 8. Interview Scenarios <a name="interview-scenarios"></a>

### Scenario 1: Reducing MTTR from 45 to 7 Minutes

**Challenge**: Production incidents had average MTTR of 45 minutes

**Solution**:
```
Phase 1: Observability Enhancement (2 weeks)
├── Deployed unified observability stack
├── Implemented distributed tracing
├── Created SLO-based alerting
└── Built automated runbooks

Phase 2: Automation (4 weeks)
├── Auto-scaling based on SLI degradation
├── Automated rollback on error spike
├── Self-healing for common issues
└── ChatOps integration (Slack bot)

Phase 3: Process Improvement (2 weeks)
├── On-call training with runbooks
├── Incident response drills
├── Post-incident review process
└── Knowledge base of past incidents

Results:
├── MTTD: 15 min → 2 min (automated detection)
├── MTTA: 10 min → 1 min (PagerDuty + mobile)
├── MTTR: 45 min → 7 min (automated mitigation)
├── Manual intervention: 80% → 25%
└── Repeat incidents: 40% → 10%
```

### Scenario 2: Implementing SLO-Based Alerting

**Challenge**: Alert fatigue with 500+ alerts per week

**Solution**:
```yaml
Before:
  alerts_per_week: 500+
  actionable_alerts: 15%
  mean_time_to_ack: 30 minutes
  team_morale: low

After:
  implementation:
    - Define clear SLOs for each service
    - Create SLO-based alerts (burn rate)
    - Remove symptom-based alerts
    - Implement tiered alerting
    
  alert_reduction:
    - Removed duplicate alerts
    - Consolidated related alerts
    - Added intelligent grouping
    
  results:
    alerts_per_week: 50
    actionable_alerts: 95%
    mean_time_to_ack: 3 minutes
    false_positive_rate: 2%
```

### Common Interview Questions

**Q1: "How do you define and measure SLOs?"**

**Answer**:
1. Start with user journeys and critical paths
2. Define SLIs that represent user experience
3. Set SLO targets based on historical data
4. Calculate error budgets (100% - SLO)
5. Implement burn rate alerting
6. Review and adjust quarterly

**Q2: "Explain your approach to reducing alert fatigue"**

**Answer**:
1. Audit existing alerts for actionability
2. Remove duplicate and low-value alerts
3. Implement SLO-based alerting
4. Use burn rate for error budget
5. Group related alerts
6. Establish clear escalation paths
7. Regular alert review process

---

## Quick Reference

```bash
# Prometheus
promtool check rules alerts.yml
promtool query instant http://prometheus:9090 'up'

# Grafana
grafana-cli plugins install grafana-piechart-panel

# Alertmanager
amtool check-config alertmanager.yml
amtool alert --alertmanager.url=http://localhost:9093

# Azure Monitor
az monitor metrics list --resource $RESOURCE_ID
az monitor log-analytics query -w $WORKSPACE_ID --analytics-query "AppRequests | take 10"

# Chaos Engineering
chaos-mesh-ctl apply network-delay.yaml
litmus chaos run pod-delete -n production
```
