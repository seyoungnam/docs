# Requests to Pod IP Addresses Do Not Work

## Issue

This example uses the following workloads in the `user-snam` namespace:

- The client Pod: `debug-with-envoy`
- The server Service: `httpbin`, backed by the `httpbin-5d7865578-swfkd` Pod

The Pod IP address for `httpbin-5d7865578-swfkd` is `100.30.10.191`.

``` bash hl_lines="5"
$ kubectl get po -n user-snam -o wide         
NAME                      READY   STATUS    RESTARTS        AGE     IP              NODE                                         NOMINATED NODE   READINESS GATES
debug-no-envoy            1/1     Running   12 (174m ago)   2d2h    100.30.22.56    ip-10-138-2-152.us-west-2.compute.internal   <none>           <none>
debug-with-envoy          2/2     Running   0               12m     100.30.5.199    ip-10-138-2-28.us-west-2.compute.internal    <none>           <none>
httpbin-5d7865578-swfkd   2/2     Running   0               46h     100.30.10.191   ip-10-138-2-104.us-west-2.compute.internal   <none>           <none>
toolbox-65c8bcc95-klz6x   2/2     Running   0               2d20h   100.30.11.97    ip-10-138-2-99.us-west-2.compute.internal    <none>           <none>
```

When the `debug-with-envoy` Pod sends a request to the `httpbin` Service by using the internal DNS name, the request succeeds.

``` bash
$ curl http://httpbin.user-snam.svc.cluster.local:80/anything
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Host": "httpbin.user-snam.svc.cluster.local", 
    "User-Agent": "curl/7.74.0", 
    "X-B3-Parentspanid": "24affb1507797e36", 
    "X-B3-Sampled": "1", 
    "X-B3-Spanid": "0a276577d31ad5eb", 
    "X-B3-Traceid": "8d2a2d831c39eade24affb1507797e36", 
    "X-Envoy-Attempt-Count": "1", 
    "X-Forwarded-Client-Cert": "By=spiffe://cluster.local/ns/user-snam/sa/default;Hash=53c78353136a3b77413c10d1f1598614642446d781ce61dfe8a8b4d87ba809b0;Subject=\"\";URI=spiffe://cluster.local/ns/user-snam/sa/default"
  }, 
  "json": null, 
  "method": "GET", 
  "origin": "127.0.0.6", 
  "url": "http://httpbin.user-snam.svc.cluster.local/anything"
}
```

However, when the `debug-with-envoy` Pod sends the same request directly to the `httpbin` Pod IP address, the connection is reset.

``` bash
$ curl http://100.30.10.191:80/anything
curl: (56) Recv failure: Connection reset by peer
```

## Why Does the Request to the Pod IP Fail?

{==In the service mesh, requests should use the service DNS name instead of the destination Pod IP address.==}

After outbound traffic is redirected to Envoy, Envoy determines the upstream endpoint in two broad phases:

- Resolve the DNS record to an IP address.
- Select the final upstream endpoint by evaluating **LISTENERS**, **ROUTES**, **CLUSTERS**, and **ENDPOINTS** in order.

When the request is made directly to the Pod IP address, the DNS resolution step is bypassed. Envoy then tries to match the original destination IP address against its listener configuration. Because `100.30.10.191` is an endpoint IP address, not the `httpbin` Service ClusterIP, there is no service-specific outbound listener for that address in the client-side Envoy configuration.

``` bash
% istioctl pc listeners debug-with-envoy --address "100.30.10.191"
ADDRESSES PORT MATCH DESTINATION
```

In this case, Envoy cannot map the request to the `httpbin.user-snam.svc.cluster.local` route, cluster, and endpoint metadata. As a result, the request does not receive the same service-mesh routing behavior as a request sent to the service DNS name, and the connection is reset. Use the service DNS name, or another Kubernetes Service that selects the intended Pod, when sending traffic through the mesh.
