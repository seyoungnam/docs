# EKS Debugging Guide

## Why Systematic Debugging?

A structured framework significantly reduces Mean Time To Recovery (MTTR).

### Unsystematic vs. Systematic

| Feature | Unsystematic | Systematic |
| :--- | :--- | :--- |
| **MTTR** | 4 hours (avg) | 30 minutes (avg) |
| **Method** | Random log search | Sequential layer inspection |
| **Approach** | Guess-based | Decision Tree-based |
| **Outcome** | Recurring problems | Prevention via Runbooks |

{==

**MTTR 8x Reduction = Systematic Framework + Repeated Training** 

==}

---

### Core Principles

1. **Classify**: Map symptom → layer.
2. **Assess Scope**: Single component vs. multiple resources.
3. **Follow Tree**: Use decision trees to narrow causes.
4. **Resolve Root**: Fix the underlying cause, not just the symptom.

---

## 6-Layer Framework

Inspect resources sequentially from top to bottom. Upper layer failures often propagate down.

<div align="center" style="margin: 20px 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 550 480" width="100%" style="max-width: 550px; height: auto;">
  <defs>
    <style>
      .box { fill: transparent; stroke-width: 2; rx: 6; ry: 6; }
      .text-title { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; font-size: 14px; font-weight: 600; fill: currentColor; }
      .text-comp { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; font-size: 13px; font-weight: 500; fill: currentColor; }
      .text-note { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; font-size: 12px; fill: #888888; }
      .arrow { stroke: currentColor; stroke-width: 1.5; stroke-dasharray: 4; }
      .arrow-solid { stroke: currentColor; stroke-width: 2; }
      .arrow-head { fill: currentColor; }
      .layer-1 { stroke: #4285f4; } /* Blue */
      .layer-2 { stroke: #f4b400; } /* Yellow */
      .layer-3 { stroke: #fb8c00; } /* Orange */
      .layer-4 { stroke: #e53935; } /* Red */
      .layer-5 { stroke: #1e88e5; } /* Blue */
      .layer-6 { stroke: #43a047; } /* Green */
    </style>
  </defs>

  <!-- Layer 1: Control Plane -->
  <rect x="20" y="20" width="140" height="45" class="box layer-1" />
  <text x="90" y="47" class="text-title" text-anchor="middle">Control Plane</text>
  <line x1="160" y1="42.5" x2="230" y2="42.5" class="arrow" />
  <polygon points="230,38.5 238,42.5 230,46.5" class="arrow-head" />
  <rect x="240" y="20" width="290" height="45" class="box layer-1" />
  <text x="385" y="47" class="text-comp" text-anchor="middle">API Server, etcd, Add-on</text>

  <line x1="90" y1="65" x2="90" y2="82" class="arrow-solid" />
  <polygon points="86,82 90,90 94,82" class="arrow-head" />

  <!-- Layer 2: Node -->
  <rect x="20" y="90" width="140" height="45" class="box layer-2" />
  <text x="90" y="117" class="text-title" text-anchor="middle">Node</text>
  <line x1="160" y1="112.5" x2="230" y2="112.5" class="arrow" />
  <polygon points="230,108.5 238,112.5 230,116.5" class="arrow-head" />
  <rect x="240" y="90" width="290" height="45" class="box layer-2" />
  <text x="385" y="117" class="text-comp" text-anchor="middle">kubelet, containerd, Karpenter</text>

  <line x1="90" y1="135" x2="90" y2="152" class="arrow-solid" />
  <polygon points="86,152 90,160 94,152" class="arrow-head" />

  <!-- Layer 3: Network -->
  <rect x="20" y="160" width="140" height="45" class="box layer-3" />
  <text x="90" y="187" class="text-title" text-anchor="middle">Network</text>
  <line x1="160" y1="182.5" x2="230" y2="182.5" class="arrow" />
  <polygon points="230,178.5 238,182.5 230,186.5" class="arrow-head" />
  <rect x="240" y="160" width="290" height="45" class="box layer-3" />
  <text x="385" y="187" class="text-comp" text-anchor="middle">VPC CNI, DNS, NetworkPolicy</text>

  <line x1="90" y1="205" x2="90" y2="222" class="arrow-solid" />
  <polygon points="86,222 90,230 94,222" class="arrow-head" />

  <!-- Layer 4: Workload -->
  <rect x="20" y="230" width="140" height="45" class="box layer-4" />
  <text x="90" y="257" class="text-title" text-anchor="middle">Workload</text>
  <line x1="160" y1="252.5" x2="230" y2="252.5" class="arrow" />
  <polygon points="230,248.5 238,252.5 230,256.5" class="arrow-head" />
  <rect x="240" y="230" width="290" height="45" class="box layer-4" />
  <text x="385" y="257" class="text-comp" text-anchor="middle">Pod, Probe, HPA, Deployment</text>

  <line x1="90" y1="275" x2="90" y2="292" class="arrow-solid" />
  <polygon points="86,292 90,300 94,292" class="arrow-head" />

  <!-- Layer 5: Storage -->
  <rect x="20" y="300" width="140" height="45" class="box layer-5" />
  <text x="90" y="327" class="text-title" text-anchor="middle">Storage</text>
  <line x1="160" y1="322.5" x2="230" y2="322.5" class="arrow" />
  <polygon points="230,318.5 238,322.5 230,326.5" class="arrow-head" />
  <rect x="240" y="300" width="290" height="45" class="box layer-5" />
  <text x="385" y="327" class="text-comp" text-anchor="middle">EBS CSI, EFS CSI, PV/PVC</text>

  <line x1="90" y1="345" x2="90" y2="362" class="arrow-solid" />
  <polygon points="86,362 90,370 94,362" class="arrow-head" />

  <!-- Layer 6: Observability -->
  <rect x="20" y="370" width="140" height="45" class="box layer-6" />
  <text x="90" y="397" class="text-title" text-anchor="middle">Observability</text>
  <line x1="160" y1="392.5" x2="230" y2="392.5" class="arrow" />
  <polygon points="230,388.5 238,392.5 230,396.5" class="arrow-head" />
  <rect x="240" y="370" width="290" height="45" class="box layer-6" />
  <text x="385" y="397" class="text-comp" text-anchor="middle">Metrics, Logs, Alerts, Dashboards</text>

  <!-- Footer Note -->
  <text x="275" y="445" class="text-note" text-anchor="middle">
    <tspan x="275" dy="0">Each layer can be debugged independently, but failures in</tspan>
    <tspan x="275" dy="20">upper layers can propagate to lower layers.</tspan>
  </text>

</svg>
</div>

1. **Control Plane**: API Server, etcd, Add-ons
2. **Node**: kubelet, containerd, Karpenter
3. **Network**: VPC CNI, DNS, NetworkPolicy
4. **Workload**: Pod, Probe, HPA, Deployment
5. **Storage**: EBS CSI, EFS CSI, PV/PVC
6. **Observability**: Metrics, Logs, Alerts, Dashboards

---

## Top-down vs. Bottom-up

Choose the debugging direction based on context.

<div align="center" style="margin: 20px 0;">
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 500 200" width="100%" style="max-width: 500px; height: auto;">
  <defs>
    <style>
      .text-label { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; font-size: 14px; font-weight: 600; fill: currentColor; }
      .text-desc { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; font-size: 12px; fill: #888888; }
      .arrow-line { stroke: currentColor; stroke-width: 2; }
      .arrow-head { fill: currentColor; }
      .bg-rect { fill: rgba(128, 128, 128, 0.05); stroke: rgba(128, 128, 128, 0.2); stroke-width: 1; rx: 8; }
    </style>
  </defs>

  <rect x="10" y="10" width="480" height="180" class="bg-rect" />

  <!-- Top Level -->
  <text x="250" y="40" class="text-label" text-anchor="middle">User / Application (Symptoms)</text>
  
  <!-- Bottom Level -->
  <text x="250" y="170" class="text-label" text-anchor="middle">Infrastructure / Control Plane (Root)</text>

  <!-- Top-down Arrow -->
  <line x1="150" y1="55" x2="150" y2="145" class="arrow-line" />
  <polygon points="145,145 150,155 155,145" class="arrow-head" />
  <text x="140" y="105" class="text-label" text-anchor="end">Top-down</text>
  <text x="140" y="120" class="text-desc" text-anchor="end">Incident Response</text>

  <!-- Bottom-up Arrow -->
  <line x1="350" y1="155" x2="350" y2="65" class="arrow-line" />
  <polygon points="345,65 350,55 355,65" class="arrow-head" />
  <text x="360" y="105" class="text-label" text-anchor="start">Bottom-up</text>
  <text x="360" y="120" class="text-desc" text-anchor="start">Health Checks</text>
</svg>
</div>

| Feature | Top-down (Symptom → Cause) | Bottom-up (Infra → App) |
| :--- | :--- | :--- |
| **Start** | Alerts / User reports | Control Plane checks |
| **Best For** | Production incidents | Preventive checks, Validations |
| **Pros** | {++Fast resolution++} | Deep visibility |
| **Cons** | {==May miss complex roots==} | Time-consuming |
| **Time** | Minutes | Hours |
| **User** | SRE / On-call | Platform / Admins |

!!! tip

    **Use Top-down for production incidents.** Find symptoms first, then drill into the specific layer.

---

## First 5 Minutes Checklist

Immediate action timeline for incident response.

- [x] **30s: Initial Diagnosis**
    * `aws eks describe-cluster`: Check cluster status
    * `kubectl get nodes`: Check node health
    * `kubectl get pods -A | grep -v Running`: Identify non-running pods
- [x] **2m: Scope Assessment**
    * Check recent events (all namespaces)
    * Aggregate Pod statuses & check distribution by Node
    * **Identify**: Single Pod? Same Node? Same AZ?
- [x] **5m: Initial Response**
    * `kubectl describe pod [name]`: Detailed pod information
    * `kubectl logs --previous`: Check previous container logs
    * `kubectl top nodes/pods`: Check resource usage

{==

**Focus on scope assessment and rapid response to minimize MTTR.**

==}