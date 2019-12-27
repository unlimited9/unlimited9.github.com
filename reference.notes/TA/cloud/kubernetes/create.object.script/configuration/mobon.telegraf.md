## configuration

#### mobon.telegraf
$ vi mobon.telegraf.yaml
```
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
