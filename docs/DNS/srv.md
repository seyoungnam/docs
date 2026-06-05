# SRV Record

## What is SRV record?

`SRV` is a DNS record type for service discovery. Unlike `A`/`AAAA` records, which answer *“what IP address is this hostname?”*, an `SRV` record answers:

> “For this service and protocol, what host and port should I connect to?”


## Format

The SRV record has the `_<service>._<protocol>.<name>` format. Example:

``` bash
_grpc._tcp.api.example.com. 300 IN SRV 10 50 8443 app-1.example.com.
_grpc._tcp.api.example.com. 300 IN SRV 10 50 8443 app-2.example.com.
```

## dnssrv

You might see an endpoint for a certain service is given as `dnssrv+_grpc._tcp.thanos-sidecar-k8s.monitoring.svc.cluster.local`. `dnssrv+` does three tasks in order:

1. Query the given `SRV` record to get the corresponding dns record and port.
2. Query `A` and/or `AAAA` records to get IP addresses.
3. Connect to(or list) those IPs using the port from the `SRV` record.

``` bash
app-1.example.com -> A/AAAA -> 10.1.2.3
connect to 10.1.2.3:8443
```

## dnssrvnoa

How is `dnssrvnoa+` different from `dnssrv+`? `dnssrvnoa+` means `DNS SRV`, no `A`/`AAAA` follow-up. `noa` is essentially **“no address lookup.”**


With `dnssrv+`:

``` bash
cache-0.cache.svc.cluster.local -> A/AAAA -> 10.2.3.4
connect/list endpoint as 10.2.3.4:11211
```

With `dnssrvnoa+`:

``` bash
keep SRV target directly
connect/list endpoint as cache-0.cache.svc.cluster.local:11211
```