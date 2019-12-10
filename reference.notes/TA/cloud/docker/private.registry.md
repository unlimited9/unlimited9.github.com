# Docker private registry

#### pull registry image
$ docker pull registry

>#### insecure registry
>--insecure-registry 설정  
>도커는 기본적으로 TLS(https)를 사용하므로 사설 저장소에 https를 설정하거나 아니면 --insecure-registry 설정을 해주고 docker daemon 제구동해야 함

#### SSL certificate
docker private registry는 보안상의 이유로 http를 지원하지 않아, localhost가 아닌 외부에서 접근시 https/ssl 인증서가 필요

ref. [self-signed certificate](../system/openssl.self.signed.certificate.md)

`generate certificate`  
$ mkdir -p /apps/docker-registry/certs  
$ openssl req \  
  -newkey rsa:4096 -nodes -sha256 -keyout /apps/docker-registry/certs/STAR.mobon.net.key \  
  -x509 -days 36500 -out /apps/docker-registry/certs/STAR.mobon.net.crt
```
Generating a 4096 bit RSA private key
.................................++
...........................................................++
writing new private key to '/apps/docker-registry/certs/STAR.mobon.net.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:KR
State or Province Name (full name) []:SEOUL
Locality Name (eg, city) [Default City]:SEOUL
Organization Name (eg, company) [Default Company Ltd]:Enliple
Organizational Unit Name (eg, section) []:Dev
Common Name (eg, your name or your server's hostname) []:*.mobon.net
Email Address []:admin@mobon.net
```
$ ls /apps/docker-registry/certs
STAR.mobon.net.crt  STAR.mobon.net.key

#### create docker private registry container (default registry port : 5000)
$ mkdir -p /apps/docker-registry/auth  
$ docker run --entrypoint htpasswd registry -Bbn mobon passwd > /apps/docker-registry/auth/htpasswd

$ mkdir -p /apps/docker-registry/volume  
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v /apps/docker-registry/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data \
  -v /apps/docker-registry/volume:/data \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v /apps/docker-registry/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/STAR.mobon.net.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/STAR.mobon.net.key \
  registry

>$ vi docker-compose.yml
>```
>version: '2'
>services:
>  docker-registry:
>    image: registry
>    restart: always
>    ports:
>      - "5000:5000"
>    volumes:
>      - /apps/docker-registry/auth:/auth
>      - /apps/docker-registry/certs:/certs
>      - /apps/docker-registry/data:/data
>    environment:
>      - REGISTRY_AUTH=htpasswd
>      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
>      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data
>      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
>      - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/STAR.mobon.net.crt
>      - REGISTRY_HTTP_TLS_KEY=/certs/STAR.mobon.net.key
>    container_name: registry
>```
>$ docker-compose up -d

>$ docker run -dit --name docker-registry -p 5000:5000 registry

#### access docker private registry
registry에 접근할 모든 client에 아래 작업을 수행한다.

`copy ca`  
docker private registry에 생성하고 등록한 인증서(STAR.mobon.net.crt)를 아래 경로에 넣어준다.
```
/etc/docker/certs.d/        <-- Certificate directory
└── localhost:5000          <-- Hostname:port
   ├── client.cert          <-- Client certificate
   ├── client.key           <-- Client key
   └── ca.crt               <-- Certificate authority that signed
                                the registry certificate
```

$ sudo mkdir -p /etc/docker/certs.d/docker-registry.mobon.net:5000  
$ sudo cp STAR.mobon.net.crt /etc/docker/certs.d/docker-registry.mobon.net:5000

>참고.
>
>`update ca`  
>* ubuntu  
>  $ cp /apps/docker-registry/certs/STAR.mobon.net.crt /usr/share/ca-certificates/  
>  $ echo /apps/docker-registry/certs/STAR.mobon.net.crt >> /etc/ca-certificates.conf  
>  $ update-ca-certificates
>
>* centos  
>  $ cp /apps/docker-registry/certs/STAR.mobon.net.crt /etc/pki/ca-trust/source/anchors/  
>  $ update-ca-trust
>
>* mac  
>  $ security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /apps/docker-registry/certs/STAR.mobon.net.crt  
>
>`restart docker`  
>$ service docker restart
>> $ systemctl restart docker.service

`host domain registration`  
ca를 사용하기 위해서 DNS나 hosts 등록이 필요

$ vi /etc/hosts
```
172.20.0.104  docker-registry.mobon.net
```

#### docker private registry login
client에서 remote docker private registry에 접근

$ docker login docker-registry.mobon.net:5000  
Username:  
Password:  
Login Succeeded

#### docker private registry image push

`tag alias`  
$ docker tag mobon/java.app.env docker-registry.mobon.net:5000/mobon/java.app.env

`push image`  
$ docker push docker-registry.mobon.net:5000/mobon/java.app.env

`get image catalog`  
$ curl -k -u 'mobon:passwd' -X GET https://docker-registry.mobon.net:5000/v2/_catalog
```
{"repositories":["mobon/java.app.env"]}
```

#### docker private registry image pull and execute
$ docker pull docker-registry.mobon.net:5000/mobon/java.app.env

## delete image

#### registry 내부의 repository 정보 조회
$ curl -k -u 'mobon:passwd' -X GET https://docker-registry.mobon.net:5000/v2/_catalog

#### repository 에 대하여 tag 정보 조회
$ curl -k -u 'mobon:passwd' -X GET https://docker-registry.mobon.net:5000/v2/mobon/java.app.env/tags/list

#### content digest 조회
$ curl -k -u 'mobon:passwd' -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X GET https://docker-registry.mobon.net:5000/v2/mobon/java.app.env/manifests/latest 2>&1 | grep Docker-Content-Digest | awk '{print ($3)}'

#### manifest 삭제
$ curl -k -u 'mobon:passwd' -v --silent -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X DELETE https://docker-registry.mobon.net:5000/v2/mobon/java.app.env/manifests/<DIGEST 정보>

#### GC(Garbage Collection)
$ docker exec -it registry_dev registry garbage-collect /etc/docker/registry/config.yml

## kubernetes

#### create a secret by providing credentials on the command line
```
kubectl create secret docker-registry <name> --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```
$ kubectl create secret docker-registry docker-reg-cred --docker-server=docker-registry.mobon.net:5000 --docker-username=mobon --docker-password=passwd --docker-email=admin@enliple.com

>$ kubectl delete secret docker-reg-cred

#### inspecting the secret docker-reg-cred
$ kubectl get secret docker-reg-cred --output=yaml
```
apiVersion: v1
data:
  .dockerconfigjson: eyJ...X0=
kind: Secret
metadata:
  creationTimestamp: "2019-09-19T04:55:41Z"
  name: docker-reg-cred
  namespace: default
  resourceVersion: "8142613"
  selfLink: /api/v1/namespaces/default/secrets/docker-reg-cred
  uid: 12ba7f05-2c0f-42d9-bb40-ac3ffc274066
type: kubernetes.io/dockerconfigjson
```

$ kubectl get secret docker-reg-cred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode
```
{"auths":{"docker-registry.mobon.net":{"username":"mobon","password":"passwd","email":"admin@enliple.com","auth":"bW9...mQ="}}}
```

$ echo "bW9...mQ=" | base64 --decode
```
mobon:passwd
```
$ cat  ~/.docker/config.json
```
{
	"auths": {
		"docker-registry.mobon.net:5000": {
			"auth": "bW9...mQ="
		}
	},
	"HttpHeaders": {
		"User-Agent": "Docker-Client/18.09.7 (linux)"
	}
}
```

#### create a pod that uses your secret
$ vi /apps/kubernetes/resources/pods/private-reg-pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: docker-reg-cred
```

$ vi /apps/kubernetes/resources/mobon.gateway.rc.yaml
```
apiVersion: v1
kind: ReplicationController
metadata:
  name: mobon-platform-gateway-aggregator-rc
spec:
  replicas: 3
  selector:
    app: mobon-platform-gateway-aggregator
  template:
    metadata:
      name: mobon-platform-gateway-aggregator-pod
      labels:
        app: mobon-platform-gateway-aggregator
    spec:
      containers:
      - name: mobon-platform-gateway-aggregator
        image: docker-registry.mobon.net:5000/mobon/java.app.env:latest
        imagePullPolicy: Always
        volumeMounts:
          - name: app-git-repository
            mountPath: /pgms/mobon.platform.gateway/repository/git
            readOnly: true
        ports:
          - containerPort: 8080
        command: ["/bin/bash", "-c"]
        args:
          - ls -al /pgms/mobon.platform.gateway/repository/git
          - gradle --build-file /pgms/mobon.platform.gateway/repository/git/aggregation.service/build.gradle :framework.boot.application:build
      imagePullSecrets:
        - name: docker-reg-cred
      volumes:
      - name: app-git-repository
        gitRepo:
          repository: http://172.20.0.7:9000/enliple/mobon/platform/gateway.git
          revision: master
          directory: .
```

## 9. Appendix

#### reference site

* Verify repository client with certificates  
https://docs.docker.com/engine/security/certificates/


+ Using a Private Registry  
https://kubernetes.io/docs/concepts/containers/images/#using-a-private-registry

+ Create a Secret based on existing Docker credentials  
https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#registry-secret-existing-credentials


- 나만의 private docker registry 구성하기  
https://novemberde.github.io/2017/04/09/Docker_Registry_0.html

- 도커 이미지 개인 저장소(Docker Private Registry) 구축  
https://m.blog.naver.com/PostView.nhn?blogId=complusblog&logNo=221000797682&proxyReferer=https%3A%2F%2Fwww.google.com%2F

- 사내 docker 저장소(registry) 구축하기  
http://www.kwangsiklee.com/2017/08/%EC%82%AC%EB%82%B4-docker-%EC%A0%80%EC%9E%A5%EC%86%8Cregistry-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/

- docker private registry  
https://setyourmindpark.github.io/2018/02/06/docker/docker-4/

- Docker registry 와 이미지 삭제  
https://trylhc.tistory.com/entry/Docker-registry-%EC%82%AD%EC%A0%9C-2
