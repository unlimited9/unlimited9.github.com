## Docker Change Port Mapping for an Existing Container

#### container information
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

#### example  
docker stop 0a9dd8ac57d0  

>sudo systemctl stop docker  

sudo vi /storage/data/docker/containers/0a9dd8ac57d037c887a415e4c626682b4c3bc1546afa410a2669f690061548b4/config.v2.json  
```
...
,"ExposedPorts":{"8080/tcp":{},"8443/tcp":{},"50750/tcp":{}},
...
,"Ports":{"8080/tcp":[{"HostIp":"","HostPort":"8080"}],"8443/tcp":[{"HostIp":"","HostPort":"8443"}],"50750/tcp":[{"HostIp":"","HostPort":"50750"}]},
...
```
sudo vi /storage/data/docker/containers/0a9dd8ac57d037c887a415e4c626682b4c3bc1546afa410a2669f690061548b4/hostconfig.json  
```
...
,"PortBindings":{"8080/tcp":[{"HostIp":"","HostPort":"8080"}],"8443/tcp":[{"HostIp":"","HostPort":"8443"}],"50750/tcp":[{"HostIp":"","HostPort":"50750"}]},
...
```
>sudo systemctl start docker  

docker start 0a9dd8ac57d0  


#### ref. How do I assign a port mapping to an existing Docker container?  
https://stackoverflow.com/questions/19335444/how-do-i-assign-a-port-mapping-to-an-existing-docker-container  
```
You can change the port mapping by directly editing the hostconfig.json file at /var/lib/docker/containers/[hash_of_the_container]/hostconfig.json

You can determine the [hash_of_the_container] via the docker inspect <container_name> command and the value of the "Id" field is the hash.

1) stop the container 
2) stop docker service (per Tacsiazuma's comment)
3) change the file
4) restart your docker engine (to flush/clear config caches)
5) start the container
So you don't need to create an image with this approach. You can also change the restart flag here.

P.S. You may visit https://docs.docker.com/engine/admin/ to learn how to correctly restart your docker engine as per your host machine. I used sudo systemctl restart docker to restart my docker engine that is running on Ubuntu 16.04
```

## 9. Appendix

#### reference site

* Docker Change Port Mapping for an Existing Container  
https://mybrainimage.wordpress.com/2017/02/05/docker-change-port-mapping-for-an-existing-container/
