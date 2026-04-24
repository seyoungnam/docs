# EKS Auto Mode

!!! info "Source Attribution"

    The primary source and original content for this debugging guide originate from the **Engineering Playbook** by DevFloor9.

    *   **Website:** [devfloor9.github.io/engineering-playbook](https://devfloor9.github.io/engineering-playbook/)
    *   **Repository:** [github.com/devfloor9/engineering-playbook](https://github.com/devfloor9/engineering-playbook)

EKS Auto Mode automates infrastructure management, shifting the debugging focus from manual configuration to AWS-managed resources.

## Auto Mode vs. Standard Mode

| Feature | Standard Mode | Auto Mode |
| :--- | :--- | :--- |
| **Node Management** | User (MNG/Karpenter) | AWS Managed (NodePool) |
| **VPC CNI** | Manual Config/Upgrade | Fully Managed |
| **GPU Driver** | GPU Operator Setup | Fully Managed |
| **Storage** | EBS CSI Separate Install | Built-in (gp3) |
| **CoreDNS** | Add-on Management | Fully Managed |
| **Node Access** | SSH Available (MNG) | Limited (SSM/Debug) |
| **Scaling** | Karpenter/CA | NodePool (Auto Spot) |

---

## Debugging & Operational Impact

<div class="grid cards" markdown>

-   :material-magnify-expand:{ .lg .middle style="color: #4285F4" } **Resource Monitoring**

    ---

    Infrastructure is abstracted. Use **NodePool CRDs** to monitor capacity health instead of individual nodes.

-   :material-lan-disconnect:{ .lg .middle style="color: #FBBC05" } **Managed Networking**

    ---

    Custom VPC CNI or CoreDNS settings are restricted. Auto Mode prioritizes stability over deep networking customization.

-   :material-chip:{ .lg .middle style="color: #EA4335" } **GPU Management**

    ---

    AWS handles drivers. If using a custom GPU Operator, you **must** disable its internal Device Plugin (`devicePlugin=false`).

-   :material-console-network:{ .lg .middle style="color: #34A853" } **Host Access**

    ---

    Direct SSH is not supported. Use **`kubectl debug node`** for all host-level troubleshooting.

-   :material-database-sync:{ .lg .middle style="color: #9C27B0" } **Storage Constraints**

    ---

    `gp3` is built-in. However, `io2 Block Express` is unsupported and EFS requires manual driver setup.

</div>

!!! tip "Hybrid Strategy"
    While Auto Mode is convenient, consider a **Hybrid Configuration** if your architecture requires specialized GPUs, high-performance storage, or custom CNI configurations.

---

## NodePool & NodeClaim

In Auto Mode, compute capacity is provisioned and managed dynamically through Custom Resource Definitions (CRDs) rather than static node groups.

<div class="grid cards" markdown>

-   :material-server-network:{ .lg .middle style="color: #FBBC05" } **NodePool CRD**

    ---

    Defines the logical node group in Auto Mode.
    Controls constraints like instance types, capacity types (Spot/On-Demand), and AZs.
    *Note:* Functionally similar to a Karpenter NodePool.

-   :material-format-list-checks:{ .lg .middle style="color: #FBBC05" } **Instance Type Selection**

    ---

    Allowed instance types are specified within the `requirements` block.
    Supports configuring fallbacks between Spot and On-Demand to ensure high availability across varied instance types.

-   :material-cube-scan:{ .lg .middle style="color: #4285F4" } **NodeClaim**

    ---

    Represents an actual 1:1 request for an EC2 instance.
    Tracks the lifecycle: `Pending` → `Launched` → `Ready`.
    Maps the physical Instance ID to the Kubernetes Node Name.

</div>

### Diagnostic Commands

**Check NodePool Status:**
```bash
# List all NodePools
kubectl get nodepools

# Inspect the default NodePool configuration and constraints
kubectl describe nodepool default
```

**Check NodeClaims (Actual Node Requests):**
```bash
# List all active NodeClaims
kubectl get nodeclaims -o wide

# Map NodeClaims directly to their provisioned Nodes
kubectl get nodeclaims -o json | \
  jq -r '.items[] | "\(.metadata.name) -> \(.status.nodeName)"'
```

**Diagnose Pending Pods (Instance Type Selection Failures):**
If a Pod is stuck in `Pending`, verify if its resource requests align with the NodePool's `requirements`.
```bash
# Check why the Pod cannot be scheduled
kubectl describe pod <pending-pod>

# Verify what instance types the NodePool is allowed to provision
kubectl get nodepool <name> -o yaml | grep -A 10 requirements
```

---

## Hybrid Configuration Best Practices

When combining EKS Auto Mode with Managed Node Groups (MNG), careful configuration is required to prevent scheduling conflicts and ensure efficient resource utilization.

### Managing Scheduling Conflicts

<div class="grid cards" markdown>

-   :material-alert-octagon:{ .lg .middle style="color: #EA4335" } **Avoid Resource Competition**

    ---

    Auto Mode NodePools and MNGs may compete for the same Pods if not properly restricted. This can lead to unpredictable scheduling behavior.
    **Solution:** Explicitly separate compute resources using **Taints and Tolerations**.

-   :material-tag-multiple:{ .lg .middle style="color: #4285F4" } **Labeling Strategy**

    ---

    Apply specific labels to MNG nodes (e.g., `workload=gpu`) and use `nodeSelector` in your Pod specifications to ensure workloads land on the correct node types.

-   :material-shield-account:{ .lg .middle style="color: #34A853" } **Taint Separation**

    ---

    Apply a taint like `nvidia.com/gpu=true:NoSchedule` to your GPU-enabled MNGs. This prevents general-purpose Pods from accidentally consuming expensive GPU resources.

-   :material-family-tree:{ .lg .middle style="color: #9C27B0" } **Hybrid Workload Pattern**

    ---

    *   **Auto Mode:** Best for elastic workloads like Web Servers, APIs, and standard Batch jobs.
    *   **Managed Node Groups:** Ideal for specialized tasks like vLLM, Training Jobs, and running the GPU Operator.

</div>

### Implementation Example: GPU MNG

When creating a dedicated node group for GPU workloads, include the appropriate taints from the start:

```bash
# Example AWS CLI command to create a tainted GPU node group
aws eks create-nodegroup \
  --cluster-name <cluster-name> \
  --nodegroup-name gpu-nodes \
  --instance-types g5.2xlarge g5.4xlarge \
  --taints "key=nvidia.com/gpu,value=true,effect=NO_SCHEDULE"
```

---

## NodeClaim Lifecycle & Monitoring

A NodeClaim goes through several phases from the moment it is requested to when the actual node is ready, and eventually when it is terminated.

### Lifecycle Phases

1.  **`Pending`**: The NodeClaim is created.
2.  **`Launched`**: EC2 instance startup begins.
3.  **`Registered`**: Kubelet registers the node with the control plane. Taints are applied.
4.  **`Initialized`**: Node is ready.
5.  **`Ready`**: The Node is fully operational and available for scheduling.

**Post-Ready States:**
*   **`Drifted`**: The node's configuration (e.g., AMI) has changed from the desired state.
*   **`Consolidation`**: The node is identified as underutilized and is targeted for replacement or removal.
*   **`Terminating`**: The node is being drained and deleted.
*   **`Deleted`**: The node and NodeClaim are removed.

### NodeClaim Status Monitoring

Use these commands to track NodeClaims through their lifecycle and identify state changes like Drift.

**Check Basic NodeClaim Status:**
```bash
kubectl get nodeclaims -o wide
# Example Output:
# NAME          TYPE         ZONE         CAPACITY   READY   AGE
# default-abc   c6i.2xlarge  us-east-1a   8          True    2d
```

**Check for Drifted Status:**
```bash
kubectl get nodeclaims -o json | jq -r '.items[] | \
  select(.status.conditions[] | select(.type=="Drifted" and .status=="True")) \
  | .metadata.name'
```

**Monitor Node Replacement in Real-Time:**
```bash
watch -n 5 'kubectl get nodeclaims -o wide'
```

---

## Troubleshooting Scheduling Failures

When Pods remain in a `Pending` state in Auto Mode (or when using Karpenter), follow this diagnostic workflow to identify why nodes are not being provisioned.

### Scheduling Failure Decision Tree

1.  **Pod Pending:** Identify the stuck workload.
2.  **Check Logs:** Inspect the provisioner logs (Karpenter/NodePool controller).
3.  **Identify Failure Type:**
    *   **`incompatible requirements`**:
        *   **Mismatch:** Pod `nodeSelector` or `affinity` conflicts with NodePool constraints.
        *   **Capacity:** Instance types or capacity types (Spot/On-Demand) are unavailable.
    *   **`instance launch failed`**:
        *   **Availability:** EC2 capacity exhaustion in the selected AZ.
        *   **Permissions:** Missing IAM permissions for the NodeRole.
    *   **`subnet/SG issues`**:
        *   **Networking:** Missing subnets or security group configurations.
        *   **Tags:** Missing `karpenter.sh/discovery` tags on subnets.

### Core Troubleshooting Commands

**Check Provisioner Logs for Launch Failures:**
```bash
# Look for launch instances or compatibility errors
kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter --tail=100 \
  | grep "launch instances\|incompatible"
```

**Compare Pod Constraints vs. NodePool Requirements:**
```bash
# Check Pod's specific scheduling requirements
kubectl get pod <pod> -o yaml | grep -A 10 "nodeSelector\|affinity"

# Check NodePool's allowed constraints
kubectl get nodepool <name> -o yaml | grep -A 20 "requirements"
```

---

## Karpenter Consolidation Mechanics

Karpenter continuously runs a consolidation loop to optimize cluster costs by removing or replacing underutilized nodes. Understanding this flow is crucial when nodes aren't scaling down as expected.

### Consolidation Decision Flow

1.  **Idle/Underutilized Node Detected:** The consolidation loop identifies a candidate node.
2.  **Can Pods be Replaced?** Karpenter checks if the pods on the node can be moved.
    *   *Check 1:* **PDB Blocked?** If `Yes` → **Postpone Consolidation**.
    *   *Check 2:* **`do-not-disrupt`?** If `Yes` → **Postpone Consolidation**.
3.  **Policy Check Passed:** If pods can be moved, it checks the NodePool's disruption policy.
4.  **Action:** Starts a new, cheaper/smaller node (if replacing) → Migrates Pods → Terminates the old node.

### Key Consolidation Blockers

If your nodes are running empty but not scaling down, check these three common blockers:

<div class="grid cards" markdown>

-   :material-shield-alert-outline:{ .lg .middle style="color: #EA4335" } **PDB Blocking**

    ---

    **Issue:** An overly strict Pod Disruption Budget (e.g., `minAvailable` is too high, leading to `allowedDisruptions: 0`).
    
    **Fix:** Use `maxUnavailable: 1` instead of high `minAvailable` percentages to ensure Karpenter is legally allowed to evict pods.

-   :material-pin-outline:{ .lg .middle style="color: #FBBC05" } **`do-not-disrupt` Annotation**

    ---

    **Issue:** Pods or NodeClaims have the `karpenter.sh/do-not-disrupt: "true"` annotation.
    
    **Use Case:** This is intentionally used to protect long-running batch jobs or critical workloads from being interrupted during consolidation.

-   :material-cog-outline:{ .lg .middle style="color: #4285F4" } **Policy Configuration**

    ---

    **Issue:** The NodePool's disruption policy dictates *how* consolidation happens.
    
    *   `WhenEmpty`: Only consolidates nodes when they are completely devoid of pods.
    *   `WhenUnderutilized`: Actively tries to bin-pack pods onto fewer/cheaper nodes. Ensure you have the right policy set for your cost vs. stability goals.

</div>

---

## Spot Interruption & Drift Handling

Managing Spot instance interruptions and configuration drift is automated in EKS Auto Mode/Karpenter, ensuring high availability and consistency.

### Spot Interruption Management

<div class="grid cards" markdown>

-   :material-clock-alert-outline:{ .lg .middle style="color: #FBBC05" } **2-Minute Warning**

    ---

    AWS issues a Spot Interruption Notice 2 minutes before termination. Karpenter **immediately** starts a replacement node to minimize downtime.

-   :material-restart:{ .lg .middle style="color: #4285F4" } **Graceful Shutdown**

    ---

    Nodes are **Cordoned** and undergo a **Graceful Shutdown**.
    **Requirement:** Set `terminationGracePeriodSeconds: 60` (recommended) to ensure pods have enough time to migrate.

-   :material-shield-sync-outline:{ .lg .middle style="color: #34A853" } **Interruption Budgets**

    ---

    Prevent mass simultaneous terminations during Spot spikes.
    **Example:** `budgets: nodes: "20%"` limits the percentage of nodes that can be interrupted at once.

</div>

### Handling Configuration Drift

Drift occurs when the actual state of a node no longer matches the desired configuration defined in the NodePool.

*   **Detection:** Karpenter detects drift in **AMI versions**, **NodePool requirements**, or **Security Group** changes.
*   **Resolution Sequence:** 
    1. Drift detected → 2. New NodeClaim created (with latest config/AMI) → 3. Pods migrated → 4. Old node terminated.
*   **Safety Control:** Use `budgets: nodes: "10%"` to limit the rate of drift-related replacements, ensuring cluster stability.

### Key Configuration Example

```yaml
spec:
  disruption:
    consolidationPolicy: WhenUnderutilized
    budgets:
    - nodes: "10%" # Limit concurrent replacements due to drift or consolidation
```

---

## Karpenter Log Analysis

Monitoring Karpenter logs provides deep insights into cluster scaling behavior, consolidation efficiency, and spot interruptions. You can analyze these logs via `kubectl` or aggregate them using CloudWatch Logs Insights.

### Key Log Patterns

Filter Karpenter logs (`kubectl logs -n karpenter -l app.kubernetes.io/name=karpenter`) for these specific keywords:

<div class="grid cards" markdown>

-   :material-check-circle-outline:{ .lg .middle style="color: #34A853" } **Provisioning Success**

    ---

    *Filter:* `grep "launched"`
    *Log Output:* `"launched nodeclaim" instance-type="c6i.2xlarge" zone="us-east-1a" capacity-type="spot"`

-   :material-alert-circle-outline:{ .lg .middle style="color: #EA4335" } **Provisioning Failure**

    ---

    *Filter:* `grep "could not launch"`
    *Log Output:* `"could not launch" err="InsufficientInstanceCapacity"`

-   :material-compress:{ .lg .middle style="color: #4285F4" } **Consolidation Execution**

    ---

    *Filter:* `grep "deprovisioning"`
    *Log Output:* `"deprovisioning via consolidation" reason="underutilized"`

-   :material-clock-alert-outline:{ .lg .middle style="color: #FBBC05" } **Spot Interruption**

    ---

    *Filter:* `grep "interruption"`
    *Log Output:* `"received spot interruption warning" time="2m"`

</div>

### CloudWatch Logs Insights Queries

If you export Karpenter logs to CloudWatch, use these queries to visualize long-term trends and cluster health.

**1. Provisioning Failure Rate by Instance Type**
```sql
fields @timestamp, instanceType, err
| filter @message like /could not launch/
| stats count() by instanceType
| sort count desc
```

**2. Number of Nodes Saved by Consolidation (Hourly)**
```sql
fields @timestamp, nodeclaim, reason
| filter @message like /deprovisioning/
| stats count() by bin(1h)
```

**3. Spot Interruption Frequency (Hourly)**
```sql
fields @timestamp, node
| filter @message like /spot interruption/
| stats count() by bin(1h)
```

**4. Node Startup Time / Provisioning Performance**
```sql
fields @timestamp, nodeclaim
| filter @message like /launched nodeclaim/
| stats avg(@duration) by instance-type
```



