# Health Check

!!! info "Source Attribution"

    The primary source and original content for this debugging guide originate from the **Engineering Playbook** by DevFloor9.

    *   **Website:** [devfloor9.github.io/engineering-playbook](https://devfloor9.github.io/engineering-playbook/)
    *   **Repository:** [github.com/devfloor9/engineering-playbook](https://github.com/devfloor9/engineering-playbook)

Independent execution of K8s Probes and Load Balancer (e.g., ALB) Health Checks can lead to mismatches and system failures.

## Common Failures

<div class="grid cards" markdown>

-   :material-server-off:{ .lg .middle style="color: #FF5277" } **503 Service Unavailable**

    ---

    **Cause:** K8s Probe succeeds, but ALB Health Check fails.

    **Details:** Often due to path mismatches or blocked Security Groups (SGs).

-   :material-link-off:{ .lg .middle style="color: #FF4081" } **502 Bad Gateway**

    ---

    **Cause:** Graceful Shutdown timing mismatch.

    **Details:** The Pod terminates, but the ALB continues routing traffic to it.

-   :material-clock-alert:{ .lg .middle style="color: #FFD740" } **Temporary 503 Error**

    ---

    **Cause:** New Pods are unready during a Rolling Update.

    **Details:** The Pod is "Ready" in K8s but hasn't passed the ALB Health Check yet.

-   :material-timer-off:{ .lg .middle style="color: #FFAB40" } **504 Timeout**

    ---

    **Cause:** Ingress timeout triggers before Backend timeout.

    **Details:** Common during large file uploads or long-running batch APIs.

</div>

---

## Health Check Mechanism Comparison

| Feature | K8s Probe | ALB Health Check | NLB Health Check | Ingress-NGINX |
| :--- | :--- | :--- | :--- | :--- |
| **Executor** | kubelet | ALB | NLB | nginx process |
| **Check From** | Internal (Node) | External → Pod IP | External → Pod IP | Internal L7 Proxy |
| **Failure Action** | Remove Endpoints | TG Deregister | TG Deregister | Remove upstream + retry |
| **Method** | HTTP/TCP/exec | HTTP(S) | TCP or HTTP | Actual traffic results |
| **Config Location** | Pod spec | Service annotation | Service annotation | Ingress annotation |

### Key Technical Differences

*   **Independent Execution:** K8s Probes and ALB Health Checks operate without knowledge of each other's status.
*   **Path Mismatch:** K8s may probe `/healthz` while the ALB defaults to `/`, leading to inconsistent health status.
*   **Timing Gap:** K8s Probes are typically fast (~10s), whereas ALB Health Checks are slower (~15-30s), creating a window where traffic may be sent to an unready Pod.

---

## Timing Matrix

Default setting comparison — where mismatches create failures:

| Setting | K8s Probe | ALB HC | NLB HC | Ingress-NGINX |
| :--- | :--- | :--- | :--- | :--- |
| **Default interval** | 10s | 15s | 30s | - (Actual traffic) |
| **Default timeout** | 1s | 5s | 6s | 60s (`proxy_read_timeout`) |
| **Failure threshold** | 3 | 2 (unhealthy) | 3 | - |
| **Success threshold** | 1 | 2 (healthy) | 3 | - |
| **Executor** | kubelet | ALB | NLB | nginx |
| **Action on failure** | Remove Endpoints | TG deregister | TG deregister | Remove upstream |
| **Check path** | `/healthz` etc. | `/` or custom | TCP or HTTP | Actual request path |

---

## Deep Dive: Mismatch Risks

### 1. The "Ghost Pod" Phenomenon (Ready vs. Healthy)
When a new Pod is deployed, K8s marks it as `Ready` almost immediately after the `readinessProbe` passes. However, the ALB requires several successful consecutive health checks (based on the `HealthyThresholdCount`) to move the target into a `Healthy` state.
*   **The Risk:** K8s might start terminating old Pods (during rolling updates) before the ALB has actually started routing traffic to the new ones, leading to a temporary capacity drop or 503 errors.

### 2. Path & Port Discrepancies
Developers often configure `readinessProbe` to a dedicated endpoint like `/healthz`, but forget to update the ALB Target Group settings.
*   **The Risk:** The app returns `200 OK` on `/healthz` (K8s is happy), but the ALB checks `/` and receives a `404 Not Found` or `403 Forbidden`. The ALB will mark the Pod as `Unhealthy` and stop routing traffic, even though the Pod is perfectly fine according to K8s.

### 3. Graceful Shutdown & Deregistration Delay
When a Pod is deleted, K8s removes it from the Endpoints list. Simultaneously, the AWS Load Balancer Controller starts the deregistration process in the ALB Target Group.
*   **The Risk:** If the ALB `deregistration_delay.timeout_seconds` (default 300s) is longer than the Pod's `terminationGracePeriodSeconds` (default 30s), the ALB may continue to send traffic to a Pod that has already stopped its application process, resulting in `502 Bad Gateway`.

### 4. Timeout Cascade
ALB timeouts and application timeouts must be carefully aligned.
*   **The Risk:** If the ALB `idle_timeout` is 60s but your Backend API takes 65s to process a request, the ALB will close the connection and return a `504 Gateway Timeout`, while your Backend continues to consume resources to finish the doomed task.

---

## Troubleshooting Pattern: Probe OK + ALB 503

When users see **503 errors** despite the Pod being **Ready** in Kubernetes, follow this decision tree to identify and resolve the issue:

1.  **Check Pod Readiness:**
    *   If **Not Ready** → Fix the `readinessProbe` configuration in the Pod spec.
2.  **Verify Endpoints:**
    *   If **Missing** → Ensure the Service selector matches the Pod labels.
3.  **Inspect Target Group (TG) Health:**
    *   If **Unhealthy**, investigate:
        *   **HC Path Mismatch?** (Most common: `readinessProbe` uses `/healthz` but ALB checks `/`) → **Fix:** Unify `healthcheck-path`.
        *   **Timeout Issues?** → **Fix:** Increase the health check timeout settings.
    *   If **Healthy**, check:
        *   **Security Groups (SG):** Ensure the Node SG allows inbound traffic from the ALB SG → **Fix:** Add SG inbound rule.

---

## Troubleshooting Pattern: 502 Bad Gateway during Shutdown

When users see **502 errors** specifically during deployments or Pod terminations, it is usually due to a timing mismatch between Pod termination and ALB deregistration.

### The Problem: Timing Mismatch
*   **T+0s:** Pod enters `Terminating` state. K8s starts the `preStop` hook, then sends `SIGTERM`.
*   **T+0s:** ALB starts the deregistration process and enters `connection draining` mode.
*   **T+1s:** The application process terminates (after receiving `SIGTERM`).
*   **T+300s (Default):** ALB completes deregistration and stops connection draining.
*   **Result:** Between T+1s and T+300s, the ALB is still draining connections and may send traffic to a Pod that has already closed its application process, causing a **502 Bad Gateway**.

### The Solution: Synchronize Timings
1.  **Reduce ALB Deregistration Delay:** Lower `deregistration_delay.timeout_seconds` to a shorter value, e.g., `15s`.
2.  **Add `preStop` Hook:** Use `sleep 15` in the Pod spec to ensure the application waits for the ALB to stop routing new traffic before it actually shuts down.
3.  **SIGTERM Handler:** Ensure the application implements a proper SIGTERM handler to finish processing existing requests before exiting.

### Shutdown Sequence Formula

To correctly calculate the required `terminationGracePeriodSeconds`, use the following formula:

**`terminationGracePeriodSeconds = deregistration_delay + preStop_sleep + app_shutdown_buffer`**

*Example Calculation:*
*   `15s` (deregistration) + `15s` (preStop sleep) + `10s` (app shutdown buffer) = **`40s`** (total grace period)

#### Recommended YAML Configuration

```yaml
spec:
  terminationGracePeriodSeconds: 40
  containers:
  - name: app
    lifecycle:
      preStop:
        exec:
          command:
          - /bin/sh
          - -c
          - |
            # Wait for ALB deregistration to be detected
            sleep 15
---
# Service annotation
metadata:
  annotations:
    alb.ingress.kubernetes.io/target-group-attributes: deregistration_delay.timeout_seconds=15
```

---

## Troubleshooting Pattern: Temporary 503 during Rolling Update

During a deployment, users may experience brief 503 errors. This is often caused by a **Timing Mismatch** between Kubernetes and the ALB.

### The Problem: Timing Gap
*   **T+10s:** K8s marks the new Pod as `Ready` (Probe succeeds).
*   **T+10s:** K8s stops sending traffic to the old Pod.
*   **T+10~30s:** The ALB Health Check has not yet passed for the new Pod.
*   **Result:** For about 20 seconds, there is no healthy target in the ALB, resulting in **503 Service Unavailable**.
*   **T+30s:** ALB Health Check passes (e.g., 2 consecutive successes) and traffic starts flowing again.

### The Solution: Deployment Controls
1.  **`minReadySeconds: 30`:** Force K8s to wait for a specific period before considering the Pod as "Ready." This should be long enough to cover the `ALB HC interval * threshold`.
2.  **`maxUnavailable: 1`:** Ensure that only a limited number of Pods are taken offline at once.
3.  **Pod Disruption Budget (PDB):** Set `minAvailable: 50%` (or similar) to guarantee a minimum level of service availability during the update.

---

## Troubleshooting Pattern: NLB Traffic Imbalance (Local Policy)

When using `externalTrafficPolicy: Local` with an NLB, you might notice uneven traffic distribution across your nodes.

### The Problem: Node-Level Health Checks
*   **Node 1:** Has 2 Pods (A, B). Health Check succeeds.
*   **Node 2:** Has 0 Pods. Health Check fails; the Node is removed from the Target Group.
*   **Node 3:** Has 1 Pod (C). Health Check succeeds.
*   **Result:** Traffic is split equally between Node 1 and Node 3. However, Node 1 receives twice as much traffic per Pod as Node 3.

### Policy Selection Guide
*   **`Local`:** Use this when you must **preserve the Client IP** and can guarantee at least 1 Pod per Node.
*   **`Cluster`:** Use this when **uniform load balancing** is the priority. Use `X-Forwarded-For` to identify client IPs if needed.

---

## Recommended Solutions

1.  **Sync Paths:** Always ensure `readinessProbe.httpGet.path` matches the ALB annotation `alb.ingress.kubernetes.io/healthcheck-path`.
2.  **Adjust Thresholds:** Lower the ALB `HealthyThresholdCount` to `2` and reduce the `IntervalSeconds` to speed up the "Ready to Healthy" transition.
3.  **Align Graceful Shutdown:** 
    *   Set `terminationGracePeriodSeconds` to be slightly longer than the ALB deregistration delay.
    *   Use a `preStop` hook to wait (e.g., `sleep 20`) to allow the ALB to stop sending new traffic before the app starts shutting down.
4.  **Security Groups:** Ensure the Node Security Group allows inbound traffic from the ALB Security Group on the application port (or the specific Health Check port).

---

## Real-World Incident

| Situation | Result | Impact |
| :--- | :--- | :--- |
| `readinessProbe`: `/healthz`<br>`ALB HC`: `/` (default) | **K8s:** Pod Ready<br>**ALB:** Unhealthy (404) | 503 Errors for users<br>**MTTR: 2 hours** (hard to debug) |


