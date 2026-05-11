# Traffic Redirection

## How Envoy and Netfilter Interact

When you inject an Envoy sidecar (like in Istio), it **leverages** Netfilter to hijack traffic:

1. **Traffic Interception**: During Pod startup, an `initContainer` (often called `istio-init`) runs a script that uses `iptables` program.
2. **Netfilter Rules**: This script adds rules to the `PREROUTING` and `OUTPUT` iptables chain.
3. **The Redirect**: These rules tell the Linux kernel: {++"If a packet is coming in for the application, don't send it to the app yet. Instead, redirect it to local port `15001` (where Envoy is listening)."++}

---

## The Core Logic (iptables script)

### 1. Inbound Redirection

This captures traffic entering the Pod and forces it into Envoy's ingress listener (usually port `15006`).

``` bash
# Create a new chain for inbound traffic
iptables -t nat -N ISTIO_INBOUND

# Redirect all TCP traffic entering the pod to the Envoy port
iptables -t nat -A ISTIO_INBOUND -p tcp -j REDIRECT --to-ports 15006

# Ensure the PREROUTING chain jumps to our new ISTIO_INBOUND chain
iptables -t nat -A PREROUTING -p tcp -j ISTIO_INBOUND
```

### 2. Outbound Redirection

This captures traffic leaving the application container and sends it to Envoy's egress listener (usually port `15001`).

``` bash
# Create a new chain for outbound traffic
iptables -t nat -N ISTIO_OUTPUT

# IMPORTANT: Don't redirect traffic that is ALREADY coming from Envoy
# (Otherwise, you get an infinite loop!)
iptables -t nat -A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN

# Redirect everything else to Envoy's egress port
iptables -t nat -A ISTIO_OUTPUT -p tcp -j REDIRECT --to-ports 15001

# Ensure the OUTPUT chain jumps to our ISTIO_OUTPUT chain
iptables -t nat -A OUTPUT -p tcp -j ISTIO_OUTPUT
```
---
## Where to find rules in a cluster

use `kubectl exec` command:

``` bash
kubectl exec [POD_NAME] -c istio-proxy -- sudo iptables -t nat -L -v
```
---
## Where is the actual code?

Check the [`istio/tools/istio-iptables/pkg`](https://github.com/istio/istio/tree/master/tools/istio-iptables/pkg) directory on Github.


---
## The Shift: Netfilter vs eBPF

The industry is moving away from `iptables` for sidecar interception for two major reasons:

1. **Latency**: Every packet has to traverse the entire Netfilter chain, which gets slower as you add more rules.
2. **Context Switching**: Moving the packet from the Kernel (Netfilter) to User Space (Envoy) and back is computationally expensive.

**The eBPF Alternative:** Modern Service Meshes (like Cilium or newer Istio versions) use **eBPF** to bypass Netfilter entirely. They **hook directly into the socket level**** to move data from the network card to Envoy much faster.