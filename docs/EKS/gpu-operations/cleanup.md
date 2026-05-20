# Cleanup

!!! Note

    The following contents are the abbreviated version of the [Generative AI on Amazon EKS](https://catalog.workshops.aws/genai-on-eks/en-US) workshop.

---
## Removing Observability Components

Let's clean up the resources we created for monitoring our LLM inference workloads:

``` bash
# Delete vLLM Service Monitor
kubectl delete servicemonitor/mistral-monitor -n monitoring

# Uninstall the DCGM Prometheus Exporter
helm uninstall dcgm-exporter -n monitoring
```

This will remove the vLLM service monitor and NVIDIA DCGM exporter that we deployed for GPU and vLLM monitoring.

---
## Cleaning up vLLM Deployment

Now let's clean up the vLLM deployment we used for monitoring:

``` bash
kubectl delete -f manifests/200-inference/vllm-s3-deployment.yml
```

Let's also remove Open WebUI:
``` bash
kubectl delete -f manifests/200-inference/openwebui.yml
```

---
## Cleaning up Grafana Ingress

``` bash
# Delete Grafana Ingress (if deployed)
kubectl delete -f manifests/300-observability/grafana-ingress.yaml 2>/dev/null || true
```

---
## Cancel On-Demand Capacity Reservation

1. Open the EC2 Console 
2. In the left navigation pane, click **Capacity Reservations**
3. Select your reservation
4. Click **Actions → Cancel**
5. Confirm the cancellation

---
## Destroy Terraform Infrastructure

Once all Kubernetes resources have been cleaned up, run the cleanup script to destroy all provisioned infrastructure:

``` bash
cd terraform
terraform destroy --auto-approve
```