# kubernetes configMap

>Created 월요일 20 11월 2017  
Container orchestration and clustering

## Getting Started

#### ConfigMap

#### create configMap
`usage`  
```
kubectl create configmap <map-name> <data-source>
```
> kubectl create configmap new-config --from-file=/somewhere --from-file=a.properties --from-literal=key=value

`create kubernetes resource file : ConfigMap`  
$ vi /apps/kubernetes/resources/mobon.gateway.git-config.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: git-config
  namespace: default
data:
  GIT_SYNC_REPO: http://172.20.0.7:9000/enliple/mobon/platform/gateway.git
  GIT_SYNC_BRANCH: master
  GIT_SYNC_ROOT: /repository/git/mobon.platform
  GIT_SYNC_DEST: 
  GIT_SYNC_PERMISSIONS: 0777
  GIT_SYNC_ONE_TIME: true
  GIT_SYNC_SSH: true
  GIT_SYNC_USERNAME: mobon_admin
  GIT_SYNC_PASSWORD: mobonproject2019!
```
>```
>---
>apiVersion: v1
>kind: ConfigMap
>metadata:
>  name: scouter-config
>  namespace: default
>data:
>  application.properties: |-
>    bean.message=Testing reload! Message from backend is: %s <br/> Services : %s  
>```

$ vi /apps/kubernetes/resources/mobon.gateway.scouter-config.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: scouter-config
  namespace: default
data:
  scouter-host.conf: |
    ### scouter host configruation sample
    net_collector_ip=172.20.0.108
    #net_collector_udp_port=6100
    #net_collector_tcp_port=6100
    #cpu_warning_pct=80
    #cpu_fatal_pct=85
    #cpu_check_period_ms=60000
    #cpu_fatal_history=3
    #cpu_alert_interval_ms=300000
    #disk_warning_pct=88
    #disk_fatal_pct=92
  scouter-java.conf: |
    ### scouter java agent configuration sample
    #obj_name=mdev-was-01-product
    # Scouter Server IP Address (Default : 127.0.0.1)
    net_collector_ip=172.20.108
    # Scouter Server Port (Default : 6100)
    #net_collector_udp_port=6100
    #net_collector_tcp_port=6100
    #hook_method_patterns=sample.mybiz.*Biz.*,sample.service.*Service.*
    trace_http_client_ip_header_key=X-Forwarded-For
    #profile_spring_controller_method_parameter_enabled=false
    #hook_exception_class_patterns=my.exception.TypedException
    #profile_fullstack_hooked_exception_enabled=true
    #hook_exception_handler_method_patterns=my.AbstractAPIController.fallbackHandler,my.ApiExceptionLoggingFilter.handleNotFoundErrorResponse
    #hook_exception_hanlder_exclude_class_patterns=exception.BizException

    ### HTTP header/parameter profile
    profile_http_parameter_enabled=true
    profile_http_header_enabled=true

    ### ignore low response time(ms)
    xlog_lower_bound_time_ms=300

    ### method profile
    #hook_method_patterns=com.xxx.controller*.*,com.xxx.service*.*,com.xxx.dao*.*
    #hook_method_ignore_classes=com.xxx.dto*.*
  config.properties: |
{{.Files.Get "static/abc/config.properties" | printf "%s" | indent 4}} # 이런식으로 패키지에 포함된 파일을 읽어서 컨피그 맵에 넣을 수도 있다. 
```

## 9. Appendix

#### reference site

* Configure a Pod to Use a ConfigMap  
https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/

* Inject Information into Pods Using a PodPreset  
https://kubernetes.io/docs/tasks/inject-data-application/podpreset/

* API Conventions  
https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#metadata

+ [Kubernetes] ConfigMap  
https://timewizhan.tistory.com/entry/Kubernetes-ConfigMap

+ [GCP]GKE 차근 차근 알아보기 4탄 — GKE 설정 외부화 (configMap, Secrets)  
https://medium.com/@jwlee98/gcp-gke-%EC%B0%A8%EA%B7%BC-%EC%B0%A8%EA%B7%BC-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-4%ED%83%84-gke-%EC%84%A4%EC%A0%95-%EC%99%B8%EB%B6%80%ED%99%94-configmap-secrets-354da3a91edf



