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
