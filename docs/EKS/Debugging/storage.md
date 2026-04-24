# Storage Debugging

!!! info "Source Attribution"

    The primary source and original content for this debugging guide originate from the **Engineering Playbook** by DevFloor9.

    *   **Website:** [devfloor9.github.io/engineering-playbook](https://devfloor9.github.io/engineering-playbook/)
    *   **Repository:** [github.com/devfloor9/engineering-playbook](https://github.com/devfloor9/engineering-playbook)

EKS storage failures often stem from EBS physical limitations and AWS API timing gaps.

## 4 Common PVC Mount Failures

<div class="grid cards" markdown>

-   :material-map-marker-alert-outline:{ .lg .middle style="color: #FBBC05" } **AZ Mismatch**

    ---

    **Cause:** EBS is locked to a single AZ. Mount fails if Pods schedule elsewhere.
    
    **Fix:** Use `volumeBindingMode: WaitForFirstConsumer` in StorageClass.

-   :material-database-alert-outline:{ .lg .middle style="color: #FBBC05" } **Volume Limit Exceeded**

    ---

    **Cause:** Instances have hard limits on attached volumes (e.g., max 128 for Nitro).
    
    **Fix:** Use larger instances or spread Pods across more nodes.

-   :material-lock-alert-outline:{ .lg .middle style="color: #FF5277" } **RWO Conflict**

    ---

    **Cause:** EBS supports only ReadWriteOnce. {== Cannot mount to multiple nodes at once. ==}
    
    **Fix:** Use EFS for `ReadWriteMany` (multi-node access) requirements.

-   :material-clock-alert-outline:{ .lg .middle style="color: #FBBC05" } **Detach Delay (~6 mins)**

    ---

    **Cause:** AWS API can take up to 6 minutes to detach a volume from a deleted Pod.
    
    **Warning:** {== **Never** force detach ==}; it risks severe data loss. Wait for graceful completion.

</div>

---

## EBS vs. EFS Comparison

Choosing the right storage type is critical for performance and availability.

| Feature | EBS (gp3) | EFS |
| :--- | :--- | :--- |
| **Access Mode** | ReadWriteOnce (RWO) | ReadWriteMany (RWX) |
| **Performance** | Up to 16,000 IOPS / 1,000 MB/s | Elastic (Bursting/Provisioned) |
| **AZ Scope** | Single AZ dependent | Multi-AZ (Auto-replication) |
| **Use Case** | DBs, StatefulSets, Single Pods | {++ Shared files, CMS, Log collection ++} |
| **Cost** | Fixed (Capacity + IOPS) | Usage-based (per GB) |
| **Constraints** | {== No multi-attach, AZ locked ==} | Requires SG TCP 2049 inbound |

### Decision Guide

*   Choose **EFS** if multiple Pods need simultaneous access to the same volume.
*   Choose **EBS (gp3)** if a single Pod requires high-performance, low-latency I/O.

---

## Storage Performance & Management

Proper configuration of StorageClasses and routine maintenance are crucial for maximizing EBS performance and avoiding unnecessary costs.

<div class="grid cards" markdown>

-   :material-speedometer:{ .lg .middle style="color: #4285F4" } **gp3 IOPS Tuning**

    ---

    **Defaults:** 3,000 IOPS / 125 MB/s.
    **Max:** Up to 16,000 IOPS / 1,000 MB/s.
    *Note:* The IOPS-to-throughput ratio must be at least 4:1.

-   :material-arrow-expand-all:{ .lg .middle style="color: #34A853" } **Volume Expansion**

    ---

    Online expansion is supported via PVC patching.
    **Requirement:** `allowVolumeExpansion: true` in the StorageClass.
    **Warning:** Volume reduction is **not** possible!

-   :material-delete-empty-outline:{ .lg .middle style="color: #FBBC05" } **Orphan Volume Cleanup**

    ---

    Manually removing PVC finalizers can leave behind orphaned EBS volumes in the `available` state.
    **Action:** Periodically check and clean these up to prevent wasted costs.

</div>

### Useful Management Commands

**Create a High-Performance gp3 StorageClass:**
```bash
cat <<EOF | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata: { name: fast-ebs }
provisioner: ebs.csi.aws.com
parameters: { type: gp3, iops: "16000", throughput: "1000" }
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
EOF
```

**Expand a Volume Online:**
```bash
kubectl patch pvc <pvc> -p '{"spec":{"resources":{"requests":{"storage":"50Gi"}}}}'
```

**Find Orphaned Kubernetes Volumes (AWS CLI):**
```bash
aws ec2 describe-volumes --filters "Name=tag:kubernetes.io/created-for/pvc/name,Values=*" \
  --query 'Volumes[?State==`available`].{ID:VolumeId,PVC:Tags[?Key==`kubernetes.io/created-for/pvc/name`]|[0].Value}'
```

---

## EKS Auto Mode Storage Constraints

EKS Auto Mode provides a built-in, managed storage experience but comes with specific support limitations and pre-configurations.

<div class="grid cards" markdown>

-   :material-check-decagram:{ .lg .middle style="color: #34A853" } **Managed EBS Driver**

    ---

    `gp3` is supported out-of-the-box without needing a manual CSI Driver installation.
    
    *   **Auto-Config:** `WaitForFirstConsumer` and `allowVolumeExpansion: true` are enabled by default.

-   :material-alert-circle-outline:{ .lg .middle style="color: #FBBC05" } **gp3 Only (gp2/io2 Limitations)**

    ---

    `gp3` is the mandatory default. `gp2` StorageClasses and `io2 Block Express` are **not supported**.
    
    *   **Encryption:** Provides default EBS encryption for all managed volumes.

-   :material-close-octagon-outline:{ .lg .middle style="color: #FF5277" } **Manual Driver Requirements**

    ---

    Advanced storage types are not managed by Auto Mode and require manual CSI Driver installation.
    
    *   **Requires Manual Setup:** EFS (for `ReadWriteMany`), FSx for Lustre, etc.

</div>

### Support Matrix

| Storage Type | Status | Note |
| :--- | :--- | :--- |
| **gp3** | :material-check: Supported | Built-in management |
| **gp2** | :material-close: Unsupported | Cannot use gp2 StorageClass |
| **io2** | :material-minus: Limited | Block Express not supported |
| **EFS / FSx** | :material-plus: Manual | Separate CSI Driver installation required |

