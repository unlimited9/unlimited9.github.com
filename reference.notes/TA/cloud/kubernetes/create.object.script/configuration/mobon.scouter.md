## configuration

#### mobon.scouter
$ vi configuration/mobon.scouter.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: scouter-config
  namespace: mobon
data:
  scouter-host.conf: |
    net_collector_ip=172.20.0.108
  scouter-java.conf: |
    net_collector_ip=172.20.108
    trace_http_client_ip_header_key=X-Forwarded-For
    profile_http_parameter_enabled=true
    profile_http_header_enabled=true
    xlog_lower_bound_time_ms=10

```

