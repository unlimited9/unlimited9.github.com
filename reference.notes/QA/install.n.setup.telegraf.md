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
  role = "grafana-server"

[agent]
  interval = "1s"
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
  hostname = "grafana-server"
  omit_hostname = false

[[outputs.influxdb]]
  urls = ["http://10.0.0.127:8086"]
  database = "telegraf"
  precision = "s"
  retention_policy = "default"
  write_consistency = "any"
  timeout = "5s"
  username = "telegraf"
  password = "telegraf"
  
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
