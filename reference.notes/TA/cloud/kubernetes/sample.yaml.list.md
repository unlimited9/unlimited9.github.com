## namespace

#### mobon.namespace
$ vi mobon.namespace.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
```

> 이후 계속 중복 정의한 Namespace 관련 부분은 계속 있다고 에러가 나지만 그냥 냅뒀다

## configuration

#### mobon.scouter.config
$ vi mobon.scouter.config.yaml
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

#### mobon.telegraf.config
$ vi mobon.telegraf.config.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: mobon
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: telegraf-config
  namespace: mobon
data:
  telegraf.conf: |
    [global_tags]
      role = "ap-server"
      scouter_obj_type_prefix = "mobon"
    
    [agent]
      interval = "5s"
      round_interval = true
      metric_batch_size = 1000
      metric_buffer_limit = 10000
      collection_jitter = "0s"
      flush_interval = "5s"
      flush_jitter = "0s"
      precision = ""
      debug = false
      quiet = false
      logfile = ""
      hostname = "ap-server"
      omit_hostname = false
    
    #[[outputs.influxdb]]
    #  urls = ["http://10.0.0.127:8086"]
    #  database = "telegraf"
    #  precision = "s"
    #  retention_policy = "default"
    #  write_consistency = "any"
    #  timeout = "5s"
    #  username = "telegraf"
    #  password = "telegraf"
    
    [[outputs.http]]
      url = "http://172.20.0.108:6100/telegraf/metric"
      timeout = "5s"
      method = "POST"
      data_format = "influx"
    
    [[inputs.statsd]]
      # Address and port to host UDP listener on
      service_address = “:8125”
      # Delete gauges every interval (default=false)
      delete_gauges = true
      # Delete counters every interval (default=false)
      delete_counters = true
      # Delete sets every interval (default=false)
      delete_sets = false
      # Delete timings & histograms every interval (default=true)
      delete_timings = true
      # Percentiles to calculate for timing & histogram stats
      percentiles = [90]
      # convert measurement names, “.” to “_” and “-” to “__”
      convert_names = false
      templates = [
        "* measurement.field"
      ]
      # Number of UDP messages allowed to queue up, once filled,
      # the statsd server will start dropping packets
      allowed_pending_messages = 10000
      # Number of timing/histogram values to track per-measurement in the
      # calculation of percentiles. Raising this limit increases the accuracy
      # of percentiles but also increases the memory usage and cpu time.
      percentile_limit = 1000
      # UDP packet size for the server to listen for. This will depend on the size
      # of the packets that the client is sending, which is usually 1500 bytes.
      udp_packet_size = 1500
    
    [[inputs.cpu]]
      percpu = true
      totalcpu = true
      collect_cpu_time = false
    
    [[inputs.disk]]
      ignore_fs = ["tmpfs", "devtmpfs"]
    
    [[inputs.kernel]]
    
    [[inputs.mem]]
    
    [[inputs.net]]
    
    [[inputs.netstat]]
    
    [[inputs.system]]
    
    [[inputs.diskio]]
    
    [[inputs.processes]]
    
```

## application

#### mobon.gateway.api
[mobon.gateway.api](create.object.script/application/mobon.gateway.api.md)

#### mobon.gateway.service
[mobon.gateway.service](create.object.script/application/mobon.gateway.service.md)

#### mobon.service.product
[mobon.service.product](create.object.script/application/mobon.service.product.md)

#### mobon.service.conversion
[mobon.service.conversion](create.object.script/application/mobon.service.conversion.md)

#### mobon.service.inclination
[mobon.service.inclination](create.object.script/application/mobon.service.inclination.md)

## solution

#### mobon.elasticsearch
[elasticsearch cluster](create.object.script/solution/mobon.elasticsearch.md)

#### mobon.redis
[redis cluster/replication](create.object.script/solution/mobon.redis.md)
