## Docker Change Port Mapping for an Existing Container

### container information
	Usage:	docker inspect [OPTIONS] NAME|ID [NAME|ID...]

Step 1: Using “docker inspect” get details about current port mapping. This will be seen under “NetworkSettings”. And “PortBindings” under “HostConfig”.

$ docker inspect mobon.elasticsearch.01

> NOTE: Stop the container and docker engine before editing the below files.

Step 2: Edit the  _config.v2.json_  file as shown below

$ vi config.v2.json

(a) Update entry for “ExposedPorts”
```
..."AttachStderr":false,"ExposedPorts":{"8080/tcp":{},"8443/tcp":{}},"Tty":...
```
(b) Update entry for “Ports”
```
..."Service":null,"Ports":{"8080/tcp":[{"HostIp":"","HostPort":"8080"}],"8443/tcp":[{"HostIp":"","HostPort":"8443"}]},"SandboxKey":...
```

Step 3: Edit the  _hostconfig.json_  file as shown below
(a) Update entry for “PortBindings”
```
...,"NetworkMode":"mobon.subnet","PortBindings":{"8080/tcp":[{"HostIp":"","HostPort":"8080"}],"8443/tcp":[{"HostIp":"","HostPort":"8443"}]},"RestartPolicy":...
```

## 9. Appendix

### reference site

#### Docker Change Port Mapping for an Existing Container
https://mybrainimage.wordpress.com/2017/02/05/docker-change-port-mapping-for-an-existing-container/
