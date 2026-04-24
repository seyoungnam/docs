# GPU Debugging

!!! info "Source Attribution"

    The primary source and original content for this debugging guide originate from the **Engineering Playbook** by DevFloor9.

    *   **Website:** [devfloor9.github.io/engineering-playbook](https://devfloor9.github.io/engineering-playbook/)
    *   **Repository:** [github.com/devfloor9/engineering-playbook](https://github.com/devfloor9/engineering-playbook)

Debugging GPU issues in EKS requires a systematic approach. The diagnostic sequence typically follows this order: **`nvidia-smi` → Driver → CUDA → Device Plugin → DCGM**.

## GPU Diagnostic Workflow

Follow this systematic flow to identify where the GPU attachment or performance is failing.

<div class="grid cards" markdown>

-   :material-console:{ .lg .middle style="color: #4285F4" } **1. Basic Driver Health**

    ---

    **Check:** Can you run `nvidia-smi` successfully?
    
    *   **Fail:** Investigate the **NVIDIA Driver Installation** on the host.

-   :material-memory:{ .lg .middle style="color: #34A853" } **2. Resource Recognition**

    ---

    **Check:** Is the GPU hardware recognized by the OS?

    *   **Fail:** Check your **ClusterPolicy** settings to ensure the hardware is being exposed to Kubernetes.

-   :material-file-code-outline:{ .lg .middle style="color: #FBBC05" } **3. Software Compatibility**

    ---

    **Focus:** Version matching between host and container.

    *   **Check:** Is the **CUDA version** compatible?
    *   **Issues:** Verify the **Driver / CUDA Matching** matrix.

-   :material-power-plug:{ .lg .middle style="color: #EA4335" } **4. K8s Integration**

    ---

    **Check:** Is the NVIDIA Device Plugin healthy and allocated?

    *   **Fail:** Re-examine the **ClusterPolicy** and ensure the plugin is running as a DaemonSet.

-   :material-chart-bell-curve-cumulative:{ .lg .middle style="color: #9C27B0" } **5. Performance Monitoring**

    ---

    **Check:** Are **DCGM metrics** being exported correctly?

    *   **Success:** Infrastructure is healthy. Proceed to **Workload-Level Debugging** for application errors.

</div>

---

## CUDA XID Error Patterns

XID errors are reported by the NVIDIA driver to the kernel log, indicating specific hardware or software failures.

| XID | Meaning | Cause | Action |
| :--- | :--- | :--- | :--- |
| **31** | GPU memory page fault | Invalid memory access | Driver update, Verify memory allocation |
| **48** | Double bit ECC error | Hardware memory defect | **Immediate Node Replacement** (Permanent) |
| **74** | NVLink error | Inter-GPU comms failure | Check NVLink topology and cables |
| **79** | GPU fallen off the bus | PCIe disconnection | **Immediate Node Replacement** (Hardware) |
| **43** | GPU stopped responding | GPU no response | Node restart required |
| **94** | Contained error | Memory integrity error | Check ECC mode, Review node replacement |

### How to Diagnose XID Errors

Search for XID errors in the kernel logs of the affected GPU node:

```bash
# Debug the node using an ephemeral container
kubectl debug node/<gpu-node> -it --image=ubuntu

# Inside the debug shell, search kernel messages
dmesg | grep -i "xid"
```

**Critical Failure Example:**
`[123.456] NVRM: Xid (PCI:0000:10:1c): 79, GPU has fallen off the bus.`
→ For **XID 48** or **XID 79**, the hardware is compromised. **Replace the node immediately.**

---

## NCCL Timeout Debugging

When performing distributed training across multiple nodes, NCCL timeouts are a common failure point. Troubleshooting these issues should prioritize verifying inter-pod communication.

### Common NCCL Timeout Causes

<div class="grid cards" markdown>

-   :material-network-outline:{ .lg .middle style="color: #4285F4" } **Multi-Node Communication**

    ---

    **Issues:** NCCL timeouts often occur during multi-node distributed training.
    
    **Action:** Verifying raw TCP/IP connectivity between Pods across different nodes is the absolute top priority.

-   :material-memory:{ .lg .middle style="color: #34A853" } **EFA Configuration**

    ---

    **Issues:** High-performance instances (e.g., p4d/p5) require Elastic Fabric Adapter (EFA).
    
    **Action:** Ensure the EFA Device Plugin is installed and `vpc.amazonaws.com/efa` resources are correctly requested in the Pod spec.

-   :material-security:{ .lg .middle style="color: #FBBC05" } **Security Group Rules**

    ---

    **Issues:** Distributed training requires extensive open communication between nodes.
    
    **Action:** Ensure the Node Security Group has a self-referencing rule allowing **all traffic** internally.

-   :material-variable:{ .lg .middle style="color: #EA4335" } **Environment Variables**

    ---

    **Issues:** Misconfigured NCCL environment variables (`NCCL_SOCKET_IFNAME`, `NCCL_IB_DISABLE`) or MPI rank mapping.
    
    **Action:** Verify that `WORLD_SIZE` and `RANK` correctly match the deployment topology.

</div>

### NCCL Debugging Configuration

To get more visibility into why NCCL is timing out, inject these environment variables into your training Pods to enable verbose logging and enforce specific network interfaces.

```yaml
env:
  - name: NCCL_DEBUG
    value: "INFO"             # Enable detailed NCCL debug logs
  - name: NCCL_DEBUG_SUBSYS
    value: "ALL"              # Log all subsystems
  - name: NCCL_SOCKET_IFNAME
    value: "eth0"             # Force NCCL to use the default VPC CNI interface
  - name: NCCL_IB_DISABLE
    value: "1"                # Disable InfiniBand (essential in standard EKS without EFA)
```

**Basic Pod-to-Pod Communication Test:**
```bash
# Test raw TCP connectivity to the target pod's training port
kubectl exec -it <pod> -- nc -zv <target-pod-ip> 12345
```

---

## vLLM Debugging & Optimization

When serving Large Language Models (LLMs) with vLLM, memory management and parallelization are key to avoiding Out-of-Memory (OOM) errors and maximizing performance.

### Key Optimization Patterns

<div class="grid cards" markdown>

-   :material-cog-outline:{ .lg .middle style="color: #4285F4" } **gpu_memory_utilization**

    ---

    **Default:** 0.9 (Uses 90% of GPU memory).
    
    *   **If OOM occurs:** Step-down to `0.85` or `0.8`.
    *   **If wasting memory:** Can be increased up to `0.95`.

-   :material-chip:{ .lg .middle style="color: #34A853" } **OOM vs KV Cache**

    ---

    *   **Model Load OOM:** Occurs when the model itself is too large. **Fix:** Use larger GPUs or Quantization (AWQ/GPTQ).
    *   **"No available blocks":** Occurs when the KV Cache is full. **Fix:** Reduce `max_model_len`.

-   :material-brain:{ .lg .middle style="color: #9C27B0" } **Tensor Parallel (TP)**

    ---

    **Best Practice:** Set `--tensor-parallel-size` equal to the number of GPUs in the Pod.
    
    *   *Example:* H100x8 (TP=8) for 70B+ models. Use powers of 2 (2, 4, 8) for `hidden_dim` alignment.

</div>

### vLLM Parameter Tuning Example

Adjust these arguments in your deployment spec to balance throughput, latency, and memory stability.

```yaml
args:
  - --model=/models/llama-3.1-70b
  - --tensor-parallel-size=4      # Match the number of assigned GPUs
  - --gpu-memory-utilization=0.85 # Reduce from default 0.9 if OOM occurs
  - --max-model-len=8192          # Determines KV Cache size
  - --max-num-batched-tokens=8192 # Balance between throughput and latency
  - --max-num-seqs=256            # Concurrent inference sequences
  - --swap-space=4                # CPU memory swap space in GiB
```

**Tuning Priority:**
If errors occur, adjust in this order: **OOM** → Reduce `gpu_memory_utilization` → Reduce `max_model_len` → Reduce `max_num_seqs`.

---

## GPU Operator Component Structure & Debugging

When relying on the NVIDIA GPU Operator, it is crucial to understand its component hierarchy. The `ClusterPolicy` is the root configuration that dictates the deployment of all other components.

### Component Dependency Flow

1.  **ClusterPolicy**
    *   Deploy → **NVIDIA Driver** → **GPU Feature Discovery**
    *   Deploy → **Container Toolkit** → **Operator Validator**
    *   Deploy → **Device Plugin** → (Registers resource `nvidia.com/gpu` to Node)
    *   Deploy → **DCGM Exporter**

### GPU Operator Diagnostic Commands

If GPUs are not being recognized or allocated, verify the health of the operator components in this specific order.

**1. Check ClusterPolicy Status**
```bash
kubectl get clusterpolicy -A
kubectl describe clusterpolicy gpu-cluster-policy
```

**2. Check All Component Pods**
```bash
kubectl get pods -n gpu-operator
```

**3. Inspect Driver Pod Logs (If installation fails)**
```bash
kubectl logs -n gpu-operator nvidia-driver-daemonset-<pod-id>
```
*   *Error `Kernel headers not found`* → Requires `kernel-devel` on the AMI.
*   *Error `nouveau driver is loaded`* → Nouveau driver must be blacklisted.

**4. Inspect Device Plugin Logs (To confirm GPU detection)**
```bash
kubectl logs -n gpu-operator nvidia-device-plugin-daemonset-<pod-id>
```
*   *Success Indicator:* `"Detected NVIDIA devices: 8"`

---

## GPU Workloads in Auto Mode

EKS Auto Mode provides a built-in GPU management system. When using it alongside traditional Managed Node Groups (MNG), specific configurations are required to avoid conflicts.

### Critical Configuration Patterns

<div class="grid cards" markdown>

-   :material-alert-circle:{ .lg .middle style="color: #EA4335" } **Disable Device Plugin**

    ---

    **Issue:** Auto Mode automatically manages GPU drivers. Enabling the GPU Operator's Device Plugin will cause a **conflict**.
    
    **Requirement:** `devicePlugin.enabled: false` must be set in the `ClusterPolicy`.

-   :material-label-off:{ .lg .middle style="color: #FBBC05" } **Isolate Node Groups**

    ---

    **Issue:** The GPU Operator should not attempt to manage Auto Mode nodes.
    
    **Requirement:** Deploy the GPU Operator exclusively to MNG nodes and use **Taints** (`nvidia.com/gpu=true:NoSchedule`) for isolation.

-   :material-layers-outline:{ .lg .middle style="color: #34A853" } **Hybrid Configuration**

    ---

    **Recommendation:** Use **Auto Mode** for general application workloads and **MNG** for dedicated, high-performance GPU tasks.
    
    **Status:** DCGM and GFD metrics collection remains fully functional in this setup.

</div>

### Hybrid Auto Mode + MNG ClusterPolicy Example

Use this configuration to allow the GPU Operator to manage MNG nodes while remaining compatible with EFA and Auto Mode.

```yaml
apiVersion: nvidia.com/v1
kind: ClusterPolicy
spec:
  driver:
    enabled: true
  devicePlugin:
    enabled: false    # CRITICAL: Disable to avoid Auto Mode conflicts
  dcgm:
    enabled: true     # Enable metric collection
  gfd:
    enabled: true     # Enable GPU Feature Discovery
```

**Node Isolation & Scheduling:**

1.  **Add Taint to MNG nodes:** `nvidia.com/gpu=true:NoSchedule`
2.  **Add Toleration to GPU Pods:** Ensure your GPU-intensive workloads have the corresponding toleration to be scheduled on the dedicated MNG nodes.

---

## GPU Diagnostic Checklist

Use this checklist for a quick, comprehensive review when troubleshooting GPU workloads.

<div class="grid cards" markdown>

-   :material-check-circle-outline:{ .lg .middle style="color: #9C27B0" } **Step 1: GPU Recognition**

    ---

    *   [ ] Check driver installation (`lsmod | grep nvidia`)
    *   [ ] Check if `ClusterPolicy` is in Ready state
    *   [ ] Check if Driver DaemonSet Pods are Running
    *   [ ] Verify the `nvidia.com/gpu.present=true` node label

-   :material-alert-circle-outline:{ .lg .middle style="color: #FBBC05" } **Step 2: Scheduling Check**

    ---

    *   [ ] Verify Device Plugin Pods are Running
    *   [ ] Check `nvidia.com/gpu` resource via `kubectl describe node`
    *   [ ] Auto Mode: Ensure `devicePlugin=false` is set
    *   [ ] Verify Pod GPU requests do not exceed Node GPU count

-   :material-brain:{ .lg .middle style="color: #FF5277" } **Step 3: vLLM OOM**

    ---

    *   [ ] Appropriate `gpu_memory_utilization` value (default 0.9)
    *   [ ] Ensure `max_model_len` is not excessively large
    *   [ ] `tensor-parallel-size` matches the number of allocated GPUs
    *   [ ] Model size fits within the available GPU memory

-   :material-access-point-network:{ .lg .middle style="color: #4285F4" } **Step 4: NCCL Timeout (Multi-Node)**

    ---

    *   [ ] SG allows all inter-node communication
    *   [ ] EFA Device Plugin is installed (for p4d/p5 instances)
    *   [ ] `NCCL_SOCKET_IFNAME` uses the correct interface
    *   [ ] `WORLD_SIZE` and `RANK` environment variables are correct

</div>




