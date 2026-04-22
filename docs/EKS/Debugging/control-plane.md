# Control Plane Debugging

## Three Major Control Plane Components

- [x] **API Server**
    *   **Role**: Entry point for all K8s requests; handles AuthN/AuthZ and Admission Control.
    *   **Failure Impact**: `kubectl` commands fail; global cluster impact.
    *   **Diagnostics**: CloudWatch `kube-apiserver-audit` logs.
- [x] **etcd**
    *   **Role**: Cluster state store (AWS-managed, no direct access).
    *   **Failure Impact**: Resource creation or modification fails.
    *   **Diagnostics**: EKS Health code `EtcdNotAvailable`.
- [x] **Add-ons**
    *   **Components**: CoreDNS, kube-proxy, VPC CNI, EBS CSI.
    *   **Failure Impact**: DNS resolution, networking, or storage operations become unavailable.
    *   **Diagnostics**: Individual add-on logs and health status.

!!! note

    As a managed service, you cannot access the control plane nodes directly. Use EKS control plane logging and the AWS Console/CLI for health status.

---

## API Server Access Failure Decision Tree

When API access fails, trace the HTTP status code or symptom to identify the root cause.

| Status / Symptom | Root Cause | Action to Take |
| :--- | :--- | :--- |
| **HTTP 401** (Unauthorized) | Authentication failure (IAM credentials issue) | Check `aws-auth` ConfigMap or Access Entries |
| **HTTP 403** (Forbidden) | Authorization failure (Insufficient RBAC) | Verify RBAC using `kubectl auth can-i` |
| **Token Expired** | SA Token reached 1-hour default limit | Upgrade AWS SDK to enable auto-renewal |
| **Timeout / No Response** | Network path blocked | Check Security Groups and VPC Endpoints |

---

## Control Plane Log Analysis

Use **CloudWatch Logs Insights** to analyze cluster health. Ensure `audit` and `authenticator` logs are enabled.

??? example "Query: API Server Errors (400+)"
    ```sql
    fields @timestamp, @message
    | filter @logStream like /kube-apiserver-audit/
    | filter responseStatus.code >= 400
    | stats count() by responseStatus.code
    | sort count desc
    ```

??? example "Query: Unauthorized Access (403)"
    ```sql
    fields @timestamp, @message
    | filter @logStream like /kube-apiserver-audit/
    | filter responseStatus.code = 403
    | stats count() by user.username
    | sort count desc
    ```

??? example "Query: Authentication Failures"
    ```sql
    fields @timestamp, @message
    | filter @logStream like /authenticator/
    | filter @message like /error/ or @message like /denied/
    | sort @timestamp desc
    ```

??? example "Query: API Throttling"
    ```sql
    fields @timestamp, @message
    | filter @logStream like /kube-apiserver/
    | filter @message like /throttle/ or @message like /rate limit/
    | stats count() by bin(5m)
    ```

---

## ServiceAccount → IAM Role Mapping

Debugging authentication issues often involves choosing between IRSA (IAM Roles for Service Accounts) and EKS Pod Identity.

### IRSA vs. Pod Identity

| Feature | IRSA | Pod Identity |
| :--- | :--- | :--- |
| **Setup** | SA annotation + OIDC Trust Policy | 1-line CLI (Association) |
| **Cross-account** | Requires sharing OIDC Provider | Simple (Role Trust only) |
| **Debugging** | Check SA annotation, Trust Policy, OIDC | Check Association existence |
| **Best For** | Legacy/Existing workloads | New workloads |

### Common Pitfalls

!!! warning "IRSA Pitfall #1: Typo in Role ARN"
    A typo in the SA annotation's `Role ARN` will silently fail, causing the Pod to **run with default node permissions instead of throwing a clear error**.

!!! warning "IRSA Pitfall #2: Namespace/SA Mismatch"
    If the `namespace` or `serviceaccount` name in the Trust Policy does not exactly match, `AssumeRole` will fail.

!!! warning "Pod Identity Pitfall: Restart Required"
    Associations are applied at Pod creation. If you add an association to an already running Pod, **a Pod restart is required** for it to take effect.

{==

**Diagnostic Command**: Run `kubectl exec [pod-name] -- env | grep AWS` to verify injected credentials.

==}

