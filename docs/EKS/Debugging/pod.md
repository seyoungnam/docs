# Pod Debugging

## Pod Status Debugging Flow

Use this guide to map Pod statuses from `kubectl get pods` to specific diagnostic actions.

| Pod Status | Primary Cause | Action to Take |
| :--- | :--- | :--- |
| **CrashLoopBackOff** | App crash after start | Check `logs`, `CMD`, `Config`, or OOM |
| **ImagePullBackOff** | Image fetch failure | Verify ECR Auth, Registry path, and Tags |
| **OOMKilled** | Memory limit exceeded | Increase `limits` or analyze memory leaks |
| **Pending** | Scheduling failure | Check Resources, Affinity, and PVC status |
| **Running 0/1 Ready** | Health check failure | Inspect `readinessProbe` configuration |
| **Terminating** | Stuck during deletion | Check `Finalizers` and `preStop` hooks |

!!! tip

    Always start with `kubectl describe pod <pod-name>` to see the **Events** section, which often contains the exact reason for the status.

---

## CrashLoopBackOff Debugging

`CrashLoopBackOff` indicates that a container is repeatedly failing and restarting.

### 4 Primary Causes

1.  **App Crash**: Uncaught exception, panic, or segfault. (Action: Check stack trace in logs).
2.  **CMD/ENTRYPOINT Error**: Incorrect execution command or path. (Action: Check Dockerfile).
3.  **Config Missing**: Missing environment variables or unmounted config files. (Action: Check ConfigMap/Secret).
4.  **OOMKilled**: Container exceeded its memory limits. (Action: Adjust `limits` or optimize app).

### Diagnostic Commands

```bash
# 1. Check previous (crashed) container logs
kubectl logs <pod-name> -n <namespace> --previous

# 2. Check Pod events (e.g., to look for OOMKilled)
kubectl describe pod <pod-name> -n <namespace>

# 3. Check specific container logs in a multi-container Pod
kubectl logs <pod-name> -c <container-name> --previous

# 4. Check the exact termination reason and exit code
kubectl get pod <pod-name> -o jsonpath='{.status.containerStatuses[0].lastState}'
```

---

## OOMKilled Deep Analysis

Understand the difference between `requests` vs `limits` and language-specific memory management strategies.

### Requests vs. Limits

| Feature | Requests | Limits |
| :--- | :--- | :--- |
| **Meaning** | Minimum guaranteed memory (for scheduling) | Maximum allowed memory (exceeding = OOMKill) |
| **Setting Recommendation** | Based on average usage (p50) | Based on peak usage (p99 + 20%) |
| **If Not Set** | **Best Effort QoS** (First to be evicted) | Unlimited (Risks **Node OOM**) |
| **QoS Impact** | **Guaranteed**: requests == limits | **Burstable**: requests < limits |

### Language-Specific Memory Management

| Runtime | Key Setting | Considerations |
| :--- | :--- | :--- |
| **JVM (Java/Kotlin)** | `-Xmx = 75% of limits` | Account for Off-heap & Metaspace. e.g. Limit 512Mi → -Xmx384m |
| **Go** | `GOMEMLIMIT` | Supported in 1.19+. Adjusts GC threshold to memory limit. |
| **Node.js** | `--max-old-space-size` | Default V8 heap is ~1.5GB. Must align with Pod limits. |
| **Python** | Memory profiling | Use `tracemalloc` or `memory_profiler` to find leaks. |

!!! danger "Critical Distinction"

    *   **Container OOMKilled**: Container exceeded its own `limits`.
    *   **Node OOM**: The sum of `requests` across all Pods exceeded total Node capacity.

---

## ImagePullBackOff Debugging

`ImagePullBackOff` or `ErrImagePull` occurs when the kubelet cannot fetch the container image.

### 3 Primary Causes

- [x] **EC2 / ECR Auth Failure**
    *   ECR tokens expire after 12 hours.
    *   Cross-account ECR access policies might be missing.
    *   Node IAM Role lacks ECR permissions.
    *   **Fix**: Ensure `ecr:GetAuthorizationToken` is attached.
- [x]  **Private Registry Issues**
    *   `imagePullSecrets` is not set on the Pod or ServiceAccount.
    *   The `docker-server` URL in the Secret is incorrect.
    *   **Fix**: `kubectl create secret docker-registry ...`
- [x]  **Image Tag Issues**
    *   The requested tag does not exist.
    *   The `:latest` tag was unexpectedly overwritten.
    *   Multi-architecture image mismatches (e.g., pulling ARM on x86).
    *   **Fix**: Use immutable tags (specific versions or SHAs).

### Diagnostic Commands

```bash
# 1. Check detailed error messages in events
kubectl describe pod <pod-name> | grep -A 5 "Events:"

# 2. Test ECR authentication locally
aws ecr get-login-password --region <region> | \
  docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com

# 3. Verify if imagePullSecrets are attached to the Pod
kubectl get pod <pod-name> -o jsonpath='{.spec.imagePullSecrets}'

# 4. Check if the image and tag actually exist in ECR
aws ecr describe-images --repository-name <repo> --image-ids imageTag=<tag>
```

---

## "Deployed but Not Working" Patterns

Cases where the Pod status is `Running`, but the service is malfunctioning.

<div class="grid cards" markdown>

-   __💔 1. Probe Failure Loop__

    ---

    *   **Status**: `Running 0/1 Ready`
    *   **Causes**: Path mismatch, insufficient `initialDelaySeconds`, or low `timeoutSeconds`.

-   __📄 2. ConfigMap/Secret Not Updated__

    ---

    *   **Issue**: Configuration changes aren't reflected in the app.
    *   **Note**: `subPath` mounts and `envFrom` do not update automatically when the source ConfigMap/Secret changes.

-   __📈 3. HPA Not Working__

    ---

    *   **Status**: `TARGETS: unknown`
    *   **Causes**: `metrics-server` missing or resource `requests` not defined (HPA requires requests to calculate percentage).

-   __🔄 4. Sidecar Termination Order__

    ---

    *   **Issue**: Request loss during Pod shutdown if the sidecar (e.g., mesh proxy) terminates before the app.
    *   **Fix**: Use K8s 1.29+ Native Sidecar support.

-   __⏰ 5. Timezone Mismatch__

    ---

    *   **Issue**: Log timestamps in UTC cause confusion.
    *   **Fix**: Set `TZ` environment variable and ensure `tzdata` is installed in the image.

-   __🚧 6. ResourceQuota Exceeded__

    ---

    *   **Issue**: `exceeded quota` errors during scaling or deployment.
    *   **Cause**: Namespace-level resource limits reached.

</div>

---

## Probe Failure Loop Debugging

`Running 0/1 Ready` indicates the Pod is running but failing health checks, preventing it from receiving traffic.

### Decision Tree for ReadinessProbe

| Question | Solution |
| :--- | :--- |
| **Path Mismatch?** | Standardize health check path (e.g., `/healthz`). |
| **Insufficient Start Time?** | Add a **startupProbe** to handle long initialization. |
| **Timeout Too Short?** | Increase `timeoutSeconds`. |
| **Actual App Failure?** | Inspect logs and dependent services. |

### Recommended Probe Configuration

Use all three probe types for robust lifecycle management (Example for Spring Boot).

```yaml
# 1. startupProbe: Verifies initial application startup
startupProbe:
  httpGet:
    path: /actuator/health
    port: 8080
  failureThreshold: 30  # Wait up to 300s (30 * 10s)
  periodSeconds: 10

# 2. readinessProbe: Determines when Pod can receive traffic
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  periodSeconds: 5
  timeoutSeconds: 3

# 3. livenessProbe: Detects deadlocks (Avoid external dependencies!)
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  periodSeconds: 10
  timeoutSeconds: 5
```

!!! warning "LivenessProbe Dependency"

    **Do not include external dependencies** (DB, Cache, APIs) in your `livenessProbe`. If a dependency goes down, the Pod will restart in a loop, potentially causing a cascading failure. Keep `livenessProbe` focused strictly on the local application state.

---

## ConfigMap / Secret Updates Not Reflected

Whether changes to a ConfigMap or Secret are automatically updated inside the Pod depends on how they are mounted.

### Mount Methods & Auto-updates

| Mount Method | Auto Update | Sync Time | Notes |
| :--- | :--- | :--- | :--- |
| **volumeMount** (Standard) | **Yes** | 1-2 mins (kubelet sync) | Recommended approach |
| **volumeMount + subPath** | No | N/A | Pod restart required |
| **envFrom / env** | No | N/A | Pod restart required |

### Mount Configuration Comparison

**🔴 Caution: subPath (No Auto-updates)**
```yaml
volumeMounts:
  - name: config
    mountPath: /etc/app/config.yaml
    subPath: config.yaml  # <--- Prevents auto-updates!
```

**🟢 Recommended: Directory Mount (Auto-updates enabled)**
```yaml
volumeMounts:
  - name: config
    mountPath: /etc/app  # <--- Mounts the entire directory
```

!!! tip

    If you must use environment variables or `subPath`, consider using [stakater/Reloader](https://github.com/stakater/Reloader) to automatically trigger a rollout restart when a ConfigMap or Secret changes.

---

## HPA Not Working

When `kubectl get hpa` shows `TARGETS: <unknown>`.

### Common Causes
1. **`metrics-server` missing**: HPA relies on the metrics-server to fetch CPU/Memory usage.
2. **Resource `requests` missing**: HPA calculates percentage based on `requests`. If not defined, it shows `<unknown>`.
3. **`minReplicas` mismatch**: If HPA `minReplicas` is greater than the Deployment's current `replicas`, scaling might not trigger as expected.

### Diagnostic Commands
```bash
# Verify HPA status
kubectl get hpa

# Verify metrics-server health (If this fails, metrics-server is the issue)
kubectl top pods
```

!!! tip "Preventing Scaling Fluctuation"

    Use `behavior.scaleDown.stabilizationWindowSeconds: 300` to prevent frequent scale-down events and maintain cluster stability.

---

## Sidecar Termination Order

Request loss often occurs when a Sidecar (e.g., Envoy, ADOT) terminates before the main application.

### The Problem
If the Sidecar exits first, the main app loses connectivity while still trying to process or finish active requests, leading to "connection refused" errors.

### Solutions

#### 1. K8s 1.29+: Native Sidecar Support
The recommended approach. Use `restartPolicy: Always` in `initContainers` to ensure the sidecar starts first and terminates last.

```yaml
initContainers:
- name: envoy
  image: envoy-proxy:latest
  restartPolicy: Always  # <--- Marks this as a Native Sidecar
containers:
- name: app              # <--- Main application
```

#### 2. Before K8s 1.28: `preStop` Hook Trick

If native support is unavailable, use `preStop` hooks to delay sidecar termination.

``` yaml
containers:
- name: app
  lifecycle:
    preStop:
      exec:
        command: ["sh", "-c", "sleep 5"]
- name: envoy
  lifecycle:
    preStop:
      exec:
        command: ["sh", "-c", "sleep 15"] # Sidecar waits longer
```

---

## Resource Quota & Timezone Issues

Two commonly overlooked pitfalls during deployment.

### 1. ResourceQuota Exceeded
When a Namespace reaches its resource limits, new Pods will fail to be created with an `exceeded quota` error.

*   **Diagnosis**: `kubectl describe resourcequota -n <namespace>`
*   **Verification**: Also check `LimitRange` settings in the namespace.

```yaml
apiVersion: v1
kind: ResourceQuota
spec:
  hard:
    requests.cpu: "100"
    requests.memory: "200Gi"
    limits.cpu: "200"
    limits.memory: "400Gi"
    pods: "100"
```

### 2. Timezone / Locale Mismatch

Containers default to UTC, which can cause log timestamp confusion.

*   **Solution**: Set the `TZ` environment variable.
*   **Requirement**: Ensure the `tzdata` package is installed (mandatory for Alpine/distroless images).

```yaml
env:
- name: TZ
  value: "Asia/Seoul"
# For JVM applications
- name: JAVA_OPTS
  value: "-Duser.timezone=Asia/Seoul"
```

---

## Rollout Strategy Comparison

Deployment behavior is controlled by the combination of `maxUnavailable` and `maxSurge`.

| Strategy | `maxUnavailable` | `maxSurge` | Behavior | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **Safety First** | `0` | `1` | Creates new Pod first, then removes old Pod. | Zero-downtime required, extra resources available. |
| **Balanced** | `1` | `1` | 1-by-1 replacement. | General workloads. |
| **Fast Deployment** | `25%` | `25%` | Simultaneous replacement (K8s Default). | Fast rollouts. |
| **Minimal Resources** | `1` | `0` | Removes old Pod first, then creates new Pod. | Resource constrained, brief downtime acceptable. |
| **Large Scale** | `10%` | `30%` | Fast scale-up + gradual removal. | Large Deployments (100+ Pods). |

### Key Rollout Parameters

!!! info "`minReadySeconds`"

    Additional wait time after a new Pod becomes Ready. Crucial for waiting until ALB Health Checks pass.
    **Recommendation**: `ALB HC interval * threshold` (e.g., `30s`).

!!! danger "`progressDeadlineSeconds`"

    The maximum time allowed for a rollout to progress (Default: `600s`). If exceeded, the Deployment status becomes `ProgressDeadlineExceeded`.
    **Rollback**: If a rollout hangs, abort and revert using `kubectl rollout undo deployment/<name>`.






