# Networking Debugging

!!! info "Source Attribution"

    The primary source and original content for this debugging guide originate from the **Engineering Playbook** by DevFloor9.

    *   **Website:** [devfloor9.github.io/engineering-playbook](https://devfloor9.github.io/engineering-playbook/)
    *   **Repository:** [github.com/devfloor9/engineering-playbook](https://github.com/devfloor9/engineering-playbook)

EKS networking follows a layered architecture. Debugging should follow a systematic workflow from the physical network layer up to the application policies.

## Networking Debugging Workflow

Follow this step-by-step diagnostic flow to identify the root cause of connectivity issues.

<div class="grid cards" markdown>

-   :material-lan-check:{ .lg .middle style="color: #4285F4" } **1. VPC CNI Layer**

    ---

    **Focus:** Pod IP allocation and ENI capacity.
    
    *   **Check:** Is the Pod IP assigned?
    *   **Issues:** IP exhaustion in subnets, ENI limits reached, or Prefix Delegation issues.

-   :material-dns:{ .lg .middle style="color: #34A853" } **2. DNS Resolution**

    ---

    **Focus:** Service discovery and external resolution.

    *   **Check:** Can the Pod resolve internal/external hostnames?
    *   **Issues:** CoreDNS OOM kills, `ndots:5` performance overhead, or lack of NodeLocal DNSCache.

-   :material-vector-link:{ .lg .middle style="color: #FBBC05" } **3. Service & Connectivity**

    ---

    **Focus:** Load balancing and service routing.

    *   **Check:** Is the Service endpoint correctly populated?
    *   **Issues:** Selector mismatches, `port` vs `targetPort` confusion, or Ingress/LB/Gateway API misconfigurations.

-   :material-shield-sync:{ .lg .middle style="color: #EA4335" } **4. NetworkPolicy**

    ---

    **Focus:** Security and access control.

    *   **Check:** Are there any policies blocking the traffic?
    *   **Issues:** Default Deny rules, "AND" vs "OR" logic confusion in selectors, or missing egress/ingress rules.

</div>

## Diagnostic Tools

For deep packet analysis and network troubleshooting, use [**netshoot**](https://hub.docker.com/r/nicolaka/netshoot).

*   **Usage:** Run as a sidecar or a standalone Pod to perform `tcpdump`, `dig`, `curl`, and other network tests from within the VPC network.

---

## Deep Dive: VPC CNI & IP Management

Issues at the VPC CNI layer often manifest as Pods stuck in the `ContainerCreating` state.

### Common VPC CNI Issues

<div class="grid cards" markdown>

-   :material-ip-network-outline:{ .lg .middle style="color: #FF5277" } **IP Exhaustion**

    ---

    **Symptom:** Pods stuck in `ContainerCreating`.
    
    Occurs when the assigned subnet has no available IP addresses left for new Pods.

-   :material-lan-disconnect:{ .lg .middle style="color: #FF4081" } **ENI Limits**

    ---

    Each EC2 instance type has a hard limit on the number of ENIs and IPs per ENI.
    
    *Example:* `c5.xlarge` supports 4 ENIs with 15 IPs each (Max 58 Pods).

-   :material-expand-all:{ .lg .middle style="color: #34A853" } **Prefix Delegation**

    ---

    Increases IP capacity by **16x** by assigning /28 prefixes to ENIs instead of individual IPs.
    
    *Example:* `c5.xlarge` capacity increases up to **110 Pods**.

</div>

### Useful Commands

**Check available IPs in a subnet:**
```bash
aws ec2 describe-subnets --subnet-ids <subnet-id> \
  --query 'Subnets[].{ID:SubnetId,AZ:AvailabilityZone,Available:AvailableIpAddressCount}'
```

**Enable Prefix Delegation:**
```bash
kubectl set env daemonset aws-node -n kube-system ENABLE_PREFIX_DELEGATION=true
```

**Check current Node allocatable Pods:**
```bash
kubectl get nodes -o json | jq '.items[] | {name: .metadata.name, allocatable_pods: .status.allocatable.pods}'
```

---

## Deep Dive: DNS Resolution

DNS issues in EKS can cause cluster-wide resolution failures or performance bottlenecks during external domain lookups.

### Common DNS Issues

<div class="grid cards" markdown>

-   :material-memory:{ .lg .middle style="color: #FF5277" } **CoreDNS OOM**

    ---

    Clusters with 5,000+ Pods often experience `OOMKilled` events during query spikes.
    
    **Solution:** Increase memory limits and implement NodeLocal DNSCache.

-   :material-dots-horizontal-circle-outline:{ .lg .middle style="color: #FF4081" } **ndots:5 Issue**

    ---

    Default K8s configuration results in 5 unnecessary queries for external domains (e.g., `api.example.com`).
    
    **Solution:** Use `ndots:2` in Pod spec or append a trailing dot to external URLs. {++ Setting `ndots:2` treats any name with 2+ dots as absolute, skipping redundant searches through cluster-local domains. (1) ++}
    { .annotate }

    1.  **The Default Behavior (`ndots: 5`)**
        When a Pod tries to resolve an external domain like `api.example.com`, the resolver counts the dots in the name. Because `api.example.com` has 2 dots, which is less than the `ndots: 5` threshold, the resolver assumes it is a _relative_ local name. 

        It will sequentially append all the Kubernetes search domains before finally trying the exact name you asked for.

        The lookup sequence looks like this:

        1. `api.example.com.default.svc.cluster.local.` (Internal - Fails)
        1. `api.example.com.svc.cluster.local.` (Internal - Fails)
        1. `api.example.com.cluster.local.` (Internal - Fails)
        1. `api.example.com.ec2.internal.` (AWS VPC default - Fails)
        1. `api.example.com.` **(Absolute query - Succeeds!)**

        When you set `ndots: 2` in the Pod spec, you lower the threshold. 

        Now, when the Pod looks up `api.example.com` (which has 2 dots), the resolver sees that the number of dots is equal to or greater than the `ndots` threshold. It immediately assumes this is an absolute domain name and tries it first.

        The lookup sequence becomes:
        1. `api.example.com.` **(Absolute query - Succeeds!)**

-   :material-cached:{ .lg .middle style="color: #34A853" } **NodeLocal DNSCache**

    ---

    Reduces CoreDNS load and overcomes VPC DNS limits (1,024 pkt/s per ENI) by caching on each node.
    
    **Essential for large-scale clusters.**

</div>

### DNS Diagnostic Commands

**Check CoreDNS Status and Performance:**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl top pods -n kube-system -l k8s-app=kube-dns
```

**Verify Pod ndots Configuration:**
```bash
kubectl exec <pod> -- cat /etc/resolv.conf
```

**Test DNS Resolution from within the Cluster:**
```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup kubernetes.default
```

---

## Deep Dive: Service Connectivity

When a service connection fails, follow this diagnostic tree to locate the issue.

### Service Diagnosis Workflow

1.  **Check Endpoints:** `kubectl get endpoints <service-name>`
2.  **Analyze Results:**
    *   **If `<none>` (Endpoints empty):**
        *   **Selector Label Mismatch:** The Service `selector` doesn't match any Pod labels.
        *   **Pod Not Ready:** The Pods match but haven't passed their readiness probes.
    *   **If IPs exist (Endpoints present):**
        *   **`port` / `targetPort` Mismatch:** The Service is sending traffic to a port the container isn't actually listening on.
        *   **Network Issues:** Blocked by `kube-proxy` rules, NetworkPolicies, or AWS Security Groups.

### Core Diagnostic Commands

**Check Endpoints:**
```bash
kubectl get endpoints <service-name>
```

**Compare Service Selector and Pod Labels:**
```bash
kubectl get svc <svc> -o jsonpath='{.spec.selector}'
kubectl get pods --show-labels
```

**Check the Actual Listening Port of a Pod:**
```bash
kubectl get pod <pod> -o jsonpath='{.spec.containers[*].ports[*].containerPort}'
```

---

## Deep Dive: NetworkPolicy Debugging

NetworkPolicy allows you to control traffic at the IP address or port level. Common pitfalls often lead to broad traffic blocks.

### Common NetworkPolicy Pitfalls

<div class="grid cards" markdown>

-   :material-lock-outline:{ .lg .middle style="color: #FF5277" } **Default Deny Risk**

    ---

    **Issues:** Using `podSelector: {}` without explicit allow rules blocks **all** ingress/egress traffic.
    
    **Solution:** Always apply "Allow" rules first before enabling a broad deny policy.

-   :material-code-brackets:{ .lg .middle style="color: #FF4081" } **AND vs OR Confusion**

    ---

    **Indentation matters.** 
    
    *   Fields in the **same** list item (same `- from` block) use **AND** logic.
    *   Fields in **separate** list items (multiple `- from` blocks) use **OR** logic.

-   :material-label-outline:{ .lg .middle style="color: #34A853" } **Missing Namespace Labels**

    ---

    `namespaceSelector` relies on labels being present on the target Namespace.
    
    **Solution:** Ensure namespaces are labeled. 
    `kubectl label ns <name> user=alice`

</div>

### Logical Operator Examples (AND vs OR)

=== "AND Logic"

    Allows only Pods with `role: client` **within** the `alice` namespace.

    ```yaml
    ingress:
    - from:
      - namespaceSelector:
          matchLabels: {user: alice}
        podSelector:
          matchLabels: {role: client}
    ```

=== "OR Logic"

    Allows **all** Pods from the `alice` namespace **OR** any Pod with `role: client` from **any** namespace.

    ```yaml
    ingress:
    - from:
      - namespaceSelector:
          matchLabels: {user: alice}
      - podSelector:
          matchLabels: {role: client}
    ```

---

## Deep Dive: Gateway API Debugging

The Gateway API introduces a more structured, role-oriented approach to routing traffic. Debugging it requires verifying the state of each resource in the chain.

### Gateway API Diagnostic Workflow

1.  **GatewayClass:** Checks controller matching and accepted conditions.
2.  **Gateway:** Verifies the `Programmed` condition (indicates Load Balancer provisioning).
3.  **HTTPRoute:** Checks `ResolvedRefs` (ensures `parentRef` and `backendRef` matching).
4.  **Service:** Final check on Endpoints health.

### Gateway API Status Check Sequence

**1. Check GatewayClass Status (Accepted condition)**
```bash
kubectl get gatewayclass
kubectl describe gatewayclass <name>
```

**2. Check Gateway Status (Programmed condition = LB provisioning)**
```bash
kubectl get gateway -A
kubectl describe gateway <name> -n <ns>
```

**3. Check HTTPRoute Status (ResolvedRefs = parentRef/backendRef matching)**
```bash
kubectl get httproute -A
kubectl describe httproute <name> -n <ns>
```

**4. Check Final Target Group Health**
```bash
aws elbv2 describe-target-health --target-group-arn <tg-arn>
```

---

## Deep Dive: netshoot Hands-on Debugging

For effective network troubleshooting, **netshoot** provides a comprehensive suite of tools (curl, dig, tcpdump, iperf3, ss, etc.) that can be used in different scenarios.

### Practical Usage Patterns

<div class="grid cards" markdown>

-   :material-bug-outline:{ .lg .middle style="color: #FF5277" } **Ephemeral Container**

    ---

    Inject a debug container directly into a running Pod without restarting it.
    
    `kubectl debug <pod> -it --image=nicolaka/netshoot`

-   :material-tools:{ .lg .middle style="color: #FF4081" } **Included Tools**

    ---

    Packed with everything needed for diagnosis: `curl`, `dig`, `tcpdump`, `iperf3`, `ss`, `traceroute`, `mtr`, `nslookup`.

-   :material-console-network-outline:{ .lg .middle style="color: #34A853" } **Standalone Debug Pod**

    ---

    Spin up a temporary Pod to test cluster-internal connectivity. Automatically deleted on exit.
    
    `kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- bash`

</div>

### Practical Troubleshooting Commands

**Check DNS resolution for a specific service:**
```bash
dig <service>.<namespace>.svc.cluster.local
```

**Test TCP connection and health endpoint:**
```bash
curl -v http://<service>.<namespace>.svc.cluster.local:<port>/health
```

**Capture packets for specific Pod IP traffic:**
```bash
tcpdump -i any host <pod-ip> -n
```

**Check socket status and connections:**
```bash
ss -tunap
```

