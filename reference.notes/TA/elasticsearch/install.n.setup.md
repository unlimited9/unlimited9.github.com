# Install and setup : Elasticsearch

>Created 목요일 30 11월 2019  
Search Engine

## 1. Pre-installation setup

#### A. creating required operating system group and user
[Create operating system group and user](../system/management.account.n.group.md)


## 8. trouble-shooting

#### Kibana server is not ready yet
```
...
FATAL Error: Index .kibana belongs to a version of Kibana that cannot be automatically migrated. Reset it or use the X-Pack upgrade assistant
...
```
$ curl -X DELETE http://elasticsearch_client_server:9200/.kibana

## 9. Appendix

#### reference site

+ Highly Available and Scalable Elasticsearch on Kubernetes  
https://medium.com/faun/https-medium-com-thakur-vaibhav23-ha-es-k8s-7e655c1b7b61
