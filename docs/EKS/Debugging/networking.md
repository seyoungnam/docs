# Networking Debugging

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

For deep packet analysis and network troubleshooting, use **netshoot**.
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
