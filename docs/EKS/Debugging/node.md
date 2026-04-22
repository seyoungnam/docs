# Node Debugging

## Node NotReady Decision Tree

When a node enters `NotReady` status, use this sequential flow to identify and resolve the issue.

| Step | Status | Action |
| :--- | :--- | :--- |
| **1. EC2 Instance** | Stopped | Restart or Replace the EC2 instance |
| **2. Kubelet** | Not Running | `systemctl restart kubelet` |
| **3. Containerd** | Not Running | `systemctl restart containerd` |
| **4. Resource Pressure** | Pressure (Disk/Mem/PID) | Evict pods or increase resource capacity |
| **5. Network** | No Response | Check SG, NACL, and VPC Routing |

!!! tip "SSM Access Required"

    Direct node access via SSM is essential for debugging. Ensure the Node IAM Role has the `AmazonSSMManagedInstanceCore` policy attached.

---

## Node Resource Pressure

Diagnose `DiskPressure`, `MemoryPressure`, and `PIDPressure`. Start by checking the node conditions:
`kubectl describe node <node-name>`

| Condition | Threshold | SSM Command | Resolution |
| :--- | :--- | :--- | :--- |
| **DiskPressure** | Available < 10% | `df -h` | `crictl rmi --prune` (Clear unused images) |
| **MemoryPressure** | Available < 100Mi | `free -m` | Evict low-priority Pods or replace node |
| **PIDPressure** | Available < 5% | `ps aux \| wc -l` | Increase `kernel.pid_max` or restart leaking container |

### Preventive Measures

1. **Monitor**: Use `eks-node-viewer` for real-time visibility.
2. **Auto-scaling**: Configure Karpenter `ttlSecondsAfterEmpty` to cycle nodes.
3. **Guardrails**: Always set appropriate resource `requests` and `limits` on workloads.

---

## EKS Auto Mode vs. Standard Mode

Debugging approaches differ based on the node management mode.

| Feature | EKS Auto Mode (Managed) | Standard Mode (Manual) |
| :--- | :--- | :--- |
| **Management** | Fully managed by AWS | Managed Node Group / Karpenter |
| **Operations** | Automated provisioning & kubelet/containerd | Direct management of services & logs |
| **Access** | No SSH/SSM access | Direct debugging via SSM |
| **Customization** | AWS optimized | Custom AMI support |
| **Debugging** | `kubectl debug node` | `SSM + journalctl` |

### Common Monitoring Tools

*   **`eks-node-viewer`**: Terminal-based real-time resource visualization.
*   **Container Insights**: CloudWatch-based infrastructure and pod metrics.
*   **Prometheus + Grafana**: Detailed custom dashboards using PromQL.

---

## EC2 Level Diagnostic Commands

Direct node debugging using SSM or `kubectl debug`.

### 1. SSM & Kubelet Debugging
```bash
# Connect via SSM
aws ssm start-session --target <instance-id>

# Check kubelet status & logs
systemctl status kubelet
journalctl -u kubelet -n 100 -f

# Check containerd status
systemctl status containerd

# Verify container runtime
crictl pods
crictl ps -a
```

### 2. Resource & Network Checks (via SSM)
```bash
# Disk & Memory
df -h
free -m

# PID count
ps aux | wc -l

# Clear unused images
crictl rmi --prune

# Node network interfaces & routes
ip addr show
ip route show
```

### 3. Debugging Without SSM (`kubectl debug`)
```bash
# Start a debug pod on the node (mounts host filesystem to /host)
kubectl debug node/<node-name> -it --image=ubuntu

# Access host filesystem
chroot /host

# Start a network debugging pod
kubectl debug node/<node-name> -it --image=nicolaka/netshoot
```

### 4. EKS Log Collector (For AWS Support)
```bash
# Download & execute log collection script
curl -O https://raw.githubusercontent.com/awslabs/amazon-eks-ami/master/log-collector-script/linux/eks-log-collector.sh
sudo bash eks-log-collector.sh

# Execute and upload directly to S3
sudo bash eks-log-collector.sh --upload s3://my-bucket/
```

!!! tip "Faster Support Resolutions"

    Always run the EKS Log Collector and attach the resulting archive when opening an AWS Support case. This drastically reduces the time needed for root cause analysis.

