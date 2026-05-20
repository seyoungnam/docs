# Observing LLM Inference workloads

!!! Note

    The following contents are the abbreviated version of the [Generative AI on Amazon EKS](https://catalog.workshops.aws/genai-on-eks/en-US) workshop.

In this section, we'll explore the core components of our observability stack designed to monitor LLM inference workloads on Amazon EKS.

---
## Observability Stack Components

For monitoring LLM inference workloads, we have deployed several key components in the `monitoring` namespace that form our observability stack:

### Kube Prometheus Stack

The Kube Prometheus Stack provides a complete monitoring solution. Let's examine each component:
``` bash
# View all components in the monitoring namespace
kubectl get pods -n monitoring

# Check Prometheus deployment
kubectl get pods -l "app.kubernetes.io/name=prometheus" -n monitoring

# Check Node Exporter DaemonSet
kubectl get pods -l "app.kubernetes.io/name=prometheus-node-exporter" -n monitoring

# Check Kube State Metrics deployment
kubectl get pods -l "app.kubernetes.io/name=kube-state-metrics" -n monitoring
```

Each component serves a specific purpose:

- **Prometheus Server**: Collects metrics from various sources and sends them to Amazon Managed Prometheus (AMP) via remote write
- **Node Exporter**: Collects hardware and OS metrics from each node (runs as DaemonSet)
- **Kube State Metrics**: Generates metrics about Kubernetes objects

### Amazon Managed Prometheus Integration

This workshop uses Amazon Managed Prometheus (AMP) for centralized metrics storage and querying. The architecture includes:

- **Prometheus Agent**: Runs in the cluster to scrape metrics from all sources
- **Remote Write**: Prometheus sends all collected metrics to AMP using AWS SigV4 authentication
- **AMP Workspace**: Stores metrics with high availability and automatic scaling
- **IAM Pod Identity**: Provides secure, automatic credential management for Prometheus and Grafana

### Grafana Stack

Our Grafana setup includes both the Grafana server and Grafana Operator to provision Dashboards using YAML files. Grafana is configured to query metrics from Amazon Managed Prometheus using IAM authentication:

``` bash
# Check Grafana Server deployment
kubectl get pods -l "app.kubernetes.io/name=grafana" -n monitoring

# Check Grafana Operator deployment
kubectl get pods -l "app.kubernetes.io/name=grafana-operator" -n monitoring
```

The Grafana instance is pre-configured with:

- **Amazon Managed Prometheus Data Source**: Set as the default data source
- **IAM Authentication**: Uses EKS Pod Identity for secure access to AMP

### Grafana Operator

Grafana Operator is being used to create Grafana dashboards using custom resources. Use the following command to check the configuration:

``` bash
kubectl get Grafana external-grafana -n monitoring -o yaml
```

### Installing NVIDIA DCGM Exporter

The [NVIDIA Data Center GPU Manager (DCGM) Exporter](https://github.com/NVIDIA/dcgm-exporter) is a tool that exposes GPU metrics for NVIDIA GPUs. It collects essential metrics such as GPU utilization, memory usage, temperature, power consumption, and other performance indicators. These metrics are exported in a Prometheus format, making it ideal for monitoring GPU workloads in Kubernetes environments.

Let's create a values file for the DCGM Exporter configuration:
``` bash
mkdir -p manifests/200-inference
cat << EOF > manifests/200-inference/values.yaml
serviceMonitor:
  enabled: true
  additionalLabels:
    release: kube-prometheus-stack  # Important for prometheus operator discovery
  interval: 30s
  honorLabels: true

service:
  enable: true
  type: ClusterIP
  port: 9400
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9400"

nodeSelector:
  karpenter.sh/nodepool: gpu

tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"

# Pod annotations for Prometheus scraping
podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "9400"

# Pod labels
podLabels:
  app.kubernetes.io/name: "dcgm-exporter"

# Resource limits to prevent OOM issues
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 256Mi

# Extra environment variables
extraEnv:
  - name: "DCGM_EXPORTER_LISTEN"
    value: ":9400"
  - name: "DCGM_EXPORTER_KUBERNETES"
    value: "true"
EOF
```

We'll now install the NVIDIA DCGM Exporter to collect GPU metrics:

``` bash
# Add NVIDIA Helm repository
helm repo add gpu-helm-charts https://nvidia.github.io/dcgm-exporter/helm-charts
helm repo update
```

``` bash
# Install DCGM Exporter in the monitoring namespace
helm install dcgm-exporter gpu-helm-charts/dcgm-exporter \
  -n monitoring \
  -f manifests/200-inference/values.yaml
```

!!! Warning "Error: INSTALLATION FAILED"

    You might encounter the following error:

    ``` bash
    Error: INSTALLATION FAILED: failed to create typed patch object (monitoring/dcgm-exporter; apps/v1, Kind=DaemonSet): errors:
      .spec.template.spec.containers[name="exporter"].env: duplicate entries for key [name="DCGM_EXPORTER_LISTEN"]
      .spec.template.spec.containers[name="exporter"].env: duplicate entries for key [name="DCGM_EXPORTER_KUBERNETES"]
    ```

    Since the chart sets these variables automatically to the exact values you've specified (`:9400` and `"true"`), you can simply delete or comment out the `extraEnv` section in your `values.yaml`:

    ``` yaml
    # Extra environment variables
    # You can safely remove or comment this out because the chart defaults to these exact values.
    # extraEnv:
    #   - name: "DCGM_EXPORTER_LISTEN"
    #     value: ":9400"
    #   - name: "DCGM_EXPORTER_KUBERNETES"
    #     value: "true"
    ```

    Completely delete the existing `dcgm-exporter` helm:

    ``` bash
    helm uninstall dcgm-exporter -n monitoring
    ```

    And retry the installating with the helm values without `extraEnv`:

    ``` bash
    helm install dcgm-exporter gpu-helm-charts/dcgm-exporter \
      -n monitoring \
      -f manifests/200-inference/values.yaml
    ```




``` bash
# Verify DCGM Exporter deployment
kubectl wait pods --for=jsonpath='{.status.phase}'=Running -l "app.kubernetes.io/name=dcgm-exporter" -n monitoring --timeout=300s
```

### Verifying GPU Metrics

In another terminal, setup the `port-forward`:

``` bash
# Get the name of a DCGM Exporter pod
NAME=$(kubectl get pods -l "app.kubernetes.io/name=dcgm-exporter" \
                       -n monitoring \
                       -o "jsonpath={ .items[0].metadata.name}")

# Set up port forwarding to access the metrics endpoint
kubectl port-forward -n monitoring $NAME 9400:9400
```

Check `/metrics` endpoint of the DCGM exporter pods to view GPU metrics.

``` bash
# Query the metrics endpoint
curl -sL http://127.0.0.1:9400/metrics

# HELP DCGM_FI_DEV_SM_CLOCK SM clock frequency (in MHz).
# TYPE DCGM_FI_DEV_SM_CLOCK gauge
DCGM_FI_DEV_SM_CLOCK{gpu="0",UUID="GPU-1c2b08c1-f602-d290-4519-863bc0c96079",pci_bus_id="00000000:30:00.0",device="nvidia0",modelName="NVIDIA L40S",Hostname="i-0e2c27f8b63f89296",container="vllm",namespace="default",pod="mistral-6cfb59d6cd-xbwgd"} 2520
# HELP DCGM_FI_DEV_MEM_CLOCK Memory clock frequency (in MHz).
# TYPE DCGM_FI_DEV_MEM_CLOCK gauge
DCGM_FI_DEV_MEM_CLOCK{gpu="0",UUID="GPU-1c2b08c1-f602-d290-4519-863bc0c96079",pci_bus_id="00000000:30:00.0",device="nvidia0",modelName="NVIDIA L40S",Hostname="i-0e2c27f8b63f89296",container="vllm",namespace="default",pod="mistral-6cfb59d6cd-xbwgd"} 9001
# HELP DCGM_FI_DEV_MEMORY_TEMP Memory temperature (in C).
# TYPE DCGM_FI_DEV_MEMORY_TEMP gauge
DCGM_FI_DEV_MEMORY_TEMP{gpu="0",UUID="GPU-1c2b08c1-f602-d290-4519-863bc0c96079",pci_bus_id="00000000:30:00.0",device="nvidia0",modelName="NVIDIA L40S",Hostname="i-0e2c27f8b63f89296",container="vllm",namespace="default",pod="mistral-6cfb59d6cd-xbwgd"} 0
```

---
## Conclusion

In this section, we have:

**Verified our existing monitoring stack components:**

- Kube Prometheus Stack
- Grafana and Grafana Operator
- Alert Manager
- Node Exporter
- Kube State Metrics

**Successfully installed NVIDIA DCGM Exporter:**

- Configured it to run only on GPU nodes
- Added proper node selectors and tolerations
- Enabled Prometheus ServiceMonitor integration

**Confirmed GPU metrics collection:**

- Verified DCGM Exporter deployment
- Accessed the metrics endpoint
- Validated GPU telemetry data