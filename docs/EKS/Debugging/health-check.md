# Health Check

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

## Real-World Incident

| Situation | Result | Impact |
| :--- | :--- | :--- |
| `readinessProbe`: `/healthz`<br>`ALB HC`: `/` (default) | **K8s:** Pod Ready<br>**ALB:** Unhealthy (404) | 503 Errors for users<br>**MTTR: 2 hours** (hard to debug) |

---

