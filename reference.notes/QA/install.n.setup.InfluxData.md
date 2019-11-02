# InfluxData 

> Real-time visibility into stacks, sensors and systems

* Telegraf : Time-Series Data Collector
* InfluxDB : Time-Series Data Storage
* Chronograf : Time-Series Data Visualization
* Kapacitor : Time-Series Data Processing

## Influxdb 
> https://docs.influxdata.com/influxdb/v1.7/introduction/installation/

#### download, install and service start
> https://portal.influxdata.com/downloads/  

$ wget https://dl.influxdata.com/influxdb/releases/influxdb_1.7.9_amd64.deb  
$ sudo dpkg -i influxdb_1.7.9_amd64.deb  
$ sudo service influxdb start

#### check process and port
$ ps -ef | grep influxdb
```
...
influxdb 14330     1 70 14:15 ?        00:00:04 /usr/bin/influxd -config /etc/influxdb/influxdb.conf
...
```
$ netstat -ntl
```
...
tcp6       0      0 :::8086                 :::*                    LISTEN     
...
```

## Telegraf

#### install
$ sudo yum install telegraf

>$ mkdir -p /apps/install  
>$ cd /apps/install  
>$ wget --no-check-certificate https://repos.influxdata.com/centos/7/x86_64/stable/telegraf-1.12.4-1.x86_64.rpm  
>$ yum localinstall telegraf-1.12.4-1.x86_64.rpm

#### run
$ serivce telegraf start

#### configuration
>$ telegraf -sample-config -input-filter cpu:disk:kernel:mem:net:netstat:system:diskio:processes -output-filter influxdb > telegraf.conf

$ vi /etc/telegraf/telegraf.conf
```
[global_tags]
  role = "ap-server"
  scouter_obj_type_prefix = "mobon.platform"

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

# APM Scouter
[[outputs.http]]
  url = "http://172.20.0.108:6180/telegraf/metric"
  timeout = "5s"
  method = "POST"
  data_format = "influx"

# Influx HTTP write listener
[[inputs.http_listener]]
  ## Address and port to host HTTP listener on
  service_address = ":8186"

  ## maximum duration before timing out read of the request
  read_timeout = "10s"
  ## maximum duration before timing out write of the response
  write_timeout = "10s"

  ## Maximum allowed http request body size in bytes.
  ## 0 means to use the default of 536,870,912 bytes (500 mebibytes)
  max_body_size = 0

  ## Maximum line size allowed to be sent in bytes.
  ## 0 means to use the default of 65536 bytes (64 kibibytes)
  max_line_size = 0

#[[inputs.statsd]]
#  # Address and port to host UDP listener on
#  service_address = ":8125"
#  # Delete gauges every interval (default=false)
#  delete_gauges = true
#  # Delete counters every interval (default=false)
#  delete_counters = true
#  # Delete sets every interval (default=false)
#  delete_sets = false
#  # Delete timings & histograms every interval (default=true)
#  delete_timings = true
#  # Percentiles to calculate for timing & histogram stats
#  percentiles = [90]
#  # convert measurement names, . to _ and - to __
#  convert_names = false
#  templates = [
#    "* measurement.field"
#  ]
#  # Number of UDP messages allowed to queue up, once filled,
#  # the statsd server will start dropping packets
#  allowed_pending_messages = 10000
#  # Number of timing/histogram values to track per-measurement in the
#  # calculation of percentiles. Raising this limit increases the accuracy
#  # of percentiles but also increases the memory usage and cpu time.
#  percentile_limit = 1000
#  # UDP packet size for the server to listen for. This will depend on the size
#  # of the packets that the client is sending, which is usually 1500 bytes.
#  udp_packet_size = 1500

#[[inputs.cpu]]
#  percpu = true
#  totalcpu = true
#  collect_cpu_time = false
#
#[[inputs.disk]]
#  ignore_fs = ["tmpfs", "devtmpfs"]
#
#[[inputs.kernel]]
#
#[[inputs.mem]]
#
#[[inputs.net]]
#
#[[inputs.netstat]]
#
#[[inputs.system]]
#
#[[inputs.diskio]]
#
#[[inputs.processes]]
#
```



## 9. Appendix

### reference site

* influxdata/Telegraf  
https://www.influxdata.com/time-series-platform/telegraf/

* influxdata/Telegraf 1.4 documentation  
https://docs.influxdata.com/telegraf/v1.4/

* influxdata/Telegraf 1.4 documentation/Aggregator and Processor Plugins  
https://docs.influxdata.com/telegraf/v1.4/concepts/aggregator_processor_plugins/

* influxdata/docs/Installing Telegraf
https://docs.influxdata.com/telegraf/v1.12/introduction/installation

* scouter-project/scouter/Telegraf Server Feature  
https://github.com/scouter-project/scouter/blob/master/scouter.document/main/Telegraf-Server_kr.md

+ Telegraf와 연동하여 Scouter에서 NGINX를 모니터링 해보자  
http://gunsdevlog.blogspot.com/2019/01/cofnguration-of-telegraf-scouter-nginx.html


- 오픈소스 시스템 모니터링 에이전트, Telegraf  
https://si.mpli.st/dev/2017-09-10-introduction-to-telegraf/

- Telegraf 설치  
https://www.mynotes.kr/telegraf-%EC%84%A4%EC%B9%98/

- grafana, influxdb, telegraf로 서버 모니터링 구축하기  
https://hero0926.tistory.com/24

- scouter-project/scouter : Telegraf Server Feature  
https://github.com/scouter-project/scouter/blob/master/scouter.document/main/Telegraf-Server.md

- Grafana; Influxdb; Telegraf; 서버의 관제 (Monitoring)와 알림 (Alerting), 그리고 자동화 하기  
https://geunhokhim.wordpress.com/2017/01/02/grafana-influxdb-telegraf-monitoring-server-alerting-automation/

- Grafana, InfluxDB 및 StatsD로 NodeJS 마이크로 서비스 앱 모니터링  
https://medium.com/@jcbaey/your-nodejs-app-deserves-grafana-influxdb-and-statsd-f61d506bdb7e

- Instrumenting Your Node/Express Application  
https://www.influxdata.com/blog/instrumenting-your-node-express-application/
