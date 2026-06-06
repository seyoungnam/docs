# Scraping Metrics Using mTLS

## Background

Prometheus scrapes workload metrics by connecting directly to Pod IP addresses. This direct endpoint access model is not compatible with Istio's sidecar proxy model. As described in [Requests to Pod IP Addresses Do Not Work](./pod-ip-not-working.md), traffic in the mesh should use service DNS names so Envoy can resolve the request through its listener, route, cluster, and endpoint configuration.

This page explains how to configure a Prometheus server Pod when it must scrape a workload by Pod IP address and the connection must use mTLS.

---

## Approach

The recommended approach is:

1. Inject an Envoy sidecar into the Prometheus server Pod.
2. Configure a certificate volume mount on the Prometheus server container.
3. Configure the Envoy sidecar to write certificates to the shared volume.
4. Configure Prometheus to use the Istiod-issued certificate files.
5. Configure the Pod so inbound and outbound traffic is not redirected to its attached Envoy sidecar. Prometheus will make direct connections to the server-side Pod using the Istiod-issued certificate.

The final step is the most important one. Prometheus must scrape the workload by Pod IP address, but direct Pod IP access does not work with Istio's normal sidecar routing model. Therefore, outbound traffic from the Prometheus server should not be redirected to its attached Envoy sidecar. On the server side, incoming traffic is still redirected to the server-side Envoy sidecar. To complete a successful TLS handshake, the Prometheus server must present an Istiod-signed certificate to the server-side Envoy sidecar.

---

## Configuration

### 1. Inject an Envoy sidecar into the Prometheus server Pod

``` bash hl_lines="5"
spec:
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "true"
```

!!! info

    - The `sidecar.istio.io/inject: "true"` annotation is not required if Envoy sidecar auto-injection is already enabled.
    - `sidecar.istio.io/inject: "true"` should be added to the **Pod** annotations, not to the **Deployment**, **StatefulSet**, or **DaemonSet** annotations.

### 2. Configure a certificate volume mount on Prometheus

``` bash
containers:
  - name: prometheus-server
    ...
    volumeMounts:
      mountPath: /etc/prom-certs/
      name: istio-certs
volumes:
  - emptyDir:
      medium: Memory
    name: istio-certs
```


### 3. Configure the Envoy sidecar to write certificates to the shared volume

``` bash hl_lines="5-8"
spec:
  template:
    metadata:
      annotations:
        proxy.istio.io/config: |  # configure the `OUTPUT_CERTS` environment variable to write certificates to the specified directory
          proxyMetadata:
            OUTPUT_CERTS: /etc/istio-output-certs
        sidecar.istio.io/userVolumeMount: '[{"name": "istio-certs", "mountPath": "/etc/istio-output-certs"}]' # mount the shared volume in the sidecar proxy
```

### 4. Configure Prometheus to use the Istiod-issued certificate

``` bash
scheme: https
tls_config:
  ca_file: /etc/prom-certs/root-cert.pem
  cert_file: /etc/prom-certs/cert-chain.pem
  key_file: /etc/prom-certs/key.pem
  insecure_skip_verify: true  # Prometheus does not support Istio security naming, so skip target Pod certificate verification
```

!!! Note

    The above configuration should be added to the Prometheus CR spec. Note there are multiple ways to add it to the Prometheus CR such as via `ConfigMap` or `Secret` object.


### 5. Disable traffic redirection to Envoy

``` bash hl_lines="5-6"
spec:
  template:
    metadata:
      annotations:
        traffic.sidecar.istio.io/includeInboundPorts: ""   # do not intercept any inbound ports
        traffic.sidecar.istio.io/includeOutboundIPRanges: ""  # do not intercept any outbound traffic
```

With this configuration, the Envoy sidecar remains idle and does not handle redirected traffic. Its primary role is to share the certificate files with the Prometheus server container.

---

## Reference

- [Prometheus](https://preliminary.istio.io/latest/docs/ops/integrations/prometheus/)
