# Observability

!!! info "Source Attribution"

    The primary source and original content for this debugging guide originate from the **Engineering Playbook** by DevFloor9.

    *   **Website:** [devfloor9.github.io/engineering-playbook](https://devfloor9.github.io/engineering-playbook/)
    *   **Repository:** [github.com/devfloor9/engineering-playbook](https://github.com/devfloor9/engineering-playbook)

EKS Observability involves a structured pipeline for collecting, analyzing, and alerting on metrics, logs, and traces from your applications, Kubernetes control plane, and infrastructure nodes.

## Observability Architecture

The architecture follows a systematic flow: **Collection → Storage/Analysis → Alerting**.

### 1. Data Sources & Collectors

<div class="grid cards" markdown>

-   :material-application-cog:{ .lg .middle style="color: #4285F4" } **Application Layer**

    ---

    **Data:** Metrics, Logs, Traces.
    **Collector:** **ADOT Collector** (AWS Distro for OpenTelemetry) facilitates high-performance telemetry ingestion.

-   :material-kubernetes:{ .lg .middle style="color: #34A853" } **Kubernetes Layer**

    ---

    **Data:** Events, K8s Metrics.
    **Collector:** **CloudWatch Agent** (Container Insights) and **Prometheus** (via `kube-state-metrics`) capture cluster-wide state.

-   :material-server-network:{ .lg .middle style="color: #FBBC05" } **Infrastructure Layer**

    ---

    **Data:** Node System Metrics (CPU, Memory, Disk).
    **Collector:** Managed agents and exporters collect host-level performance data.

</div>

### 2. Storage & Analysis

Once collected, the data is routed to specialized storage and visualization platforms:

*   **CloudWatch Logs & Metrics:** Centralized repository for log aggregation and AWS-native metric storage.
*   **Amazon Managed Prometheus (AMP):** Highly available, scalable Prometheus-compatible monitoring service.
*   **Grafana Dashboards:** The primary visualization layer for creating rich, interactive dashboards from AMP and CloudWatch data.

### 3. Alerting Pipeline

The final stage ensures critical issues are communicated to the right teams:

*   **CloudWatch Alarms:** Triggered based on threshold breaches in CloudWatch metrics.
*   **Alertmanager:** Manages alerts from Prometheus, handling silencing, inhibition, and aggregation.
*   **Notifications:** Routing to **SNS, PagerDuty, or Slack** for real-time incident response.

---

## Container Insights & Application Signals

AWS provides advanced monitoring capabilities through Container Insights and Application Signals, offering deep visibility into cluster performance and application health.

### Core Monitoring Capabilities

<div class="grid cards" markdown>

-   :material-chart-bell-curve-cumulative:{ .lg .middle style="color: #34A853" } **Enhanced Monitoring**

    ---

    **Collector:** CloudWatch Agent automatically collects Node, Pod, and Container-level metrics.
    **Key Metrics:** CPU, Memory, Network, and Disk I/O.
    **Logs:** Integrates with Fluent Bit for comprehensive log collection.

-   :material-eye-outline:{ .lg .middle style="color: #4285F4" } **Application Signals**

    ---

    **Feature:** ADOT-based automatic distributed tracing.
    **Integration:** Combines X-Ray traces with CloudWatch metrics.
    **Benefits:** Automatically calculates Service Maps, Latency, and Error Rates without code changes.

-   :material-bullseye-arrow:{ .lg .middle style="color: #FBBC05" } **SLI/SLO Based Alerting**

    ---

    **Target:** Define SLOs (e.g., 99.9% Availability).
    **Measurement:** Tracks SLIs based on Latency P99 and Error Rates.
    **Triggers:** Alerts are fired based on Error Budget burn rates, reducing alert fatigue.

</div>

### Setup & Core PromQL Queries

**Install the Container Insights Add-on:**
```bash
aws eks create-addon --cluster-name $CLUSTER \
  --addon-name amazon-cloudwatch-observability
```

**Detect CPU Throttling (over 25% indicates performance degradation):**
```promql
sum(rate(container_cpu_cfs_throttled_periods_total{namespace="prod"}[5m]))
/ sum(rate(container_cpu_cfs_periods_total{namespace="prod"}[5m])) > 0.25
```

**Detect OOMKilled Pods:**
```promql
kube_pod_container_status_last_terminated_reason{reason="OOMKilled"} > 0
```

**Node Memory Warning (exceeding 85%):**
```promql
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 85
```

---

## Incident Detection Patterns

Effective incident response relies on diverse detection strategies to minimize time-to-detect (MTTD) while avoiding alert fatigue.

### 4 Common Detection Patterns

| Pattern | Method | Pros | Limitations | Tools |
| :--- | :--- | :--- | :--- | :--- |
| **Threshold-based** | Alerts on pre-defined value breaches. | Simple & Predictable | High False Positives | CloudWatch Alarms, Prometheus |
| **Anomaly Detection** | ML-based "normal" pattern learning. | Dynamic scaling support | Needs 2 weeks training | CloudWatch Anomaly Detection |
| **Composite Alarms** | AND/OR logic across multiple alarms. | Drastic FP reduction | Complex configuration | CloudWatch Composite Alarms |
| **Log Metric Filters**| Logic-to-metric pattern matching. | Custom event detection | Real-time delay | CloudWatch Metric Filters |

### Incident Detection Maturity Model

The maturity of your observability stack is measured by how quickly and automatically you can identify and resolve failures.

<div class="grid cards" markdown>

-   :material-numeric-1-circle-outline:{ .lg .middle style="color: #9E9E9E" } **L1: Basic**

    ---

    **MTTD:** < 30 minutes
    **Method:** Manual monitoring and reactive troubleshooting.

-   :material-numeric-2-circle-outline:{ .lg .middle style="color: #4285F4" } **L2: Standard**

    ---

    **MTTD:** < 10 minutes
    **Method:** Static thresholds and basic log-based alerting.

-   :material-numeric-3-circle-outline:{ .lg .middle style="color: #FBBC05" } **L3: Advanced**

    ---

    **MTTD:** < 5 minutes
    **Method:** Anomaly detection and logical composite alarms.

-   :material-numeric-4-circle-outline:{ .lg .middle style="color: #34A853" } **L4: Automated**

    ---

    **MTTD:** < 1 minute
    **Method:** Fully automated detection paired with self-healing recovery actions.

</div>

---

## Alert Optimization & Preventing Alert Fatigue

When operations teams receive too many alerts, they begin to ignore them—a dangerous state known as Alert Fatigue. Alerts must be actionable, prioritized, and targeted.

### Strategies for Alert Optimization

<div class="grid cards" markdown>

-   :material-bell-off-outline:{ .lg .middle style="color: #FF5277" } **Preventing Alert Fatigue**

    ---

    *   **Filter the Noise:** Only send P3/P4 level alerts to Slack channels.
    *   **Targeted Escalation:** Only route P1/P2 critical alerts to PagerDuty.
    *   **Maintenance:** Periodically review and tune alert rules.

-   :material-filter-variant:{ .lg .middle style="color: #34A853" } **Utilizing Composite Alarms**

    ---

    Combine multiple signals to detect *actual* incidents rather than symptoms.
    *   `Error Rate Spikes` **AND** `Latency Increases` = Service Disruption.
    *   `Error Rate Spikes` **AND** `Pod Restarts` = Application Crash loop.

</div>

### Recommended Alert Channel Matrix

Standardizing alert routing ensures the right response time for the right severity.

| Severity | Alert Channel | Response SLA | Example Scenario |
| :--- | :--- | :--- | :--- |
| <span style="color:#FF5277">**P1 Critical**</span> | PagerDuty + Phone Call | 15 Minutes | Complete Service Outage |
| <span style="color:#FBBC05">**P2 High**</span> | Slack DM + PagerDuty | 30 Minutes | Partial Outage, Severe Degradation |
| <span style="color:#4285F4">**P3 Medium**</span> | Slack Channel | 4 Hours | Pod Restarts, High Resource Warnings |
| <span style="color:#9E9E9E">**P4 Low**</span> | Email / Jira | Next Business Day | Disk Usage Growing, Cert Expiring Soon |

