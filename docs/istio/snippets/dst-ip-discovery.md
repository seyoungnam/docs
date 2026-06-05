# Destination IP Address Discovery

## Background

This example uses the following workloads in the `user-snam` namespace:

- The client Pod: `debug-with-envoy`
- The server Service: `httpbin`, backed by the `httpbin-5d7865578-swfkd` Pod

``` bash hl_lines="5" title="Pods in user-snam namespace"
$ kubectl get po -n user-snam -o wide         
NAME                      READY   STATUS    RESTARTS        AGE     IP              NODE                                         NOMINATED NODE   READINESS GATES
debug-no-envoy            1/1     Running   12 (174m ago)   2d2h    100.30.22.56    ip-10-138-2-152.us-west-2.compute.internal   <none>           <none>
debug-with-envoy          2/2     Running   0               12m     100.30.5.199    ip-10-138-2-28.us-west-2.compute.internal    <none>           <none>
httpbin-5d7865578-swfkd   2/2     Running   0               46h     100.30.10.191   ip-10-138-2-104.us-west-2.compute.internal   <none>           <none>
toolbox-65c8bcc95-klz6x   2/2     Running   0               2d20h   100.30.11.97    ip-10-138-2-99.us-west-2.compute.internal    <none>           <none>
```

``` bash title="Service in user-snam namespace"
% k get svc
NAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
httpbin   ClusterIP   172.20.159.220   <none>        80/TCP    134d
```

This walkthrough explains how a request sent from the `debug-with-envoy` Pod is resolved from the service DNS name to the final destination Pod IP address.

## xDS Configuration

After outbound traffic is redirected from the application container to Envoy, Envoy determines the upstream endpoint in two broad phases:

- Resolve the DNS record to an IP address.
- Select the final upstream endpoint by evaluating **LISTENERS**, **ROUTES**, **CLUSTERS**, and **ENDPOINTS** in order.

The request starts with the internal DNS name. First, `httpbin.user-snam.svc.cluster.local` resolves to `172.20.159.220`, which is the ClusterIP for the `httpbin` Service. The listener configuration contains an entry for `172.20.159.220:80`, which points to the `httpbin.user-snam.svc.cluster.local:80` route.

``` bash hl_lines="5 6"
$ istioctl pc listeners debug-with-envoy 
ADDRESSES      PORT  MATCH                                                   DESTINATION
172.20.116.236 80    Trans: raw_buffer; App: http/1.1,h2c                    Route: istio-ingressgateway-default-nlb.istio-gateway.svc.cluster.local:80
172.20.116.236 80    ALL                                                     Cluster: outbound|80||istio-ingressgateway-default-nlb.istio-gateway.svc.cluster.local
172.20.159.220 80    Trans: raw_buffer; App: http/1.1,h2c                    Route: httpbin.user-snam.svc.cluster.local:80
172.20.159.220 80    ALL                                                     Cluster: outbound|80||httpbin.user-snam.svc.cluster.local
172.20.16.141  80    Trans: raw_buffer; App: http/1.1,h2c                    Route: istio-ingressgateway-internal-nlb.istio-gateway.svc.cluster.local:80
172.20.16.141  80    ALL                                                     Cluster: outbound|80||istio-ingressgateway-internal-nlb.istio-gateway.svc.cluster.local
```

That route maps to the `httpbin.user-snam.svc.cluster.local:80` virtual host.

``` bash hl_lines="3"
$ istioctl pc routes debug-with-envoy --name httpbin.user-snam.svc.cluster.local:80
NAME                                       VHOST NAME                                 DOMAINS     MATCH     VIRTUAL SERVICE
httpbin.user-snam.svc.cluster.local:80     httpbin.user-snam.svc.cluster.local:80     *           /*        
```

The cluster output shows that the `httpbin.user-snam.svc.cluster.local:80` virtual host resolves to the `outbound|80||httpbin.user-snam.svc.cluster.local` cluster.

``` bash hl_lines="3"
% istioctl pc clusters debug-with-envoy --fqdn httpbin.user-snam.svc.cluster.local
SERVICE FQDN                            PORT     SUBSET     DIRECTION     TYPE     DESTINATION RULE
httpbin.user-snam.svc.cluster.local     80       -          outbound      EDS      httpbin-dr.user-snam
```

Using that cluster name, the endpoint configuration for the `debug-with-envoy` Pod shows the final upstream endpoint IP address.

``` bash hl_lines="3"
% istioctl pc endpoints debug-with-envoy --cluster "outbound|80||httpbin.user-snam.svc.cluster.local"
ENDPOINT             STATUS      OUTLIER CHECK     CLUSTER
100.30.10.191:80     HEALTHY     OK                outbound|80||httpbin.user-snam.svc.cluster.local
```

The endpoint IP address, `100.30.10.191`, matches the IP address of the `httpbin-5d7865578-swfkd` Pod.

``` bash hl_lines="5" title="Pods in user-snam namespace"
$ kubectl get po -n user-snam -o wide         
NAME                      READY   STATUS    RESTARTS        AGE     IP              NODE                                         NOMINATED NODE   READINESS GATES
debug-no-envoy            1/1     Running   12 (174m ago)   2d2h    100.30.22.56    ip-10-138-2-152.us-west-2.compute.internal   <none>           <none>
debug-with-envoy          2/2     Running   0               12m     100.30.5.199    ip-10-138-2-28.us-west-2.compute.internal    <none>           <none>
httpbin-5d7865578-swfkd   2/2     Running   0               46h     100.30.10.191   ip-10-138-2-104.us-west-2.compute.internal   <none>           <none>
toolbox-65c8bcc95-klz6x   2/2     Running   0               2d20h   100.30.11.97    ip-10-138-2-99.us-west-2.compute.internal    <none>           <none>
```
