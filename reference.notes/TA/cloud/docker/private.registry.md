# Docker private registry

#### pull registry image
$ docker pull registry

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
server.crt  server.key

`update ca`  
* ubuntu  
  $ cp /apps/docker-registry/certs/STAR.mobon.net.crt /usr/share/ca-certificates/  
  $ echo /apps/docker-registry/certs/STAR.mobon.net.crt >> /etc/ca-certificates.conf  
  $ update-ca-certificates

* centos  
  $ cp /apps/docker-registry/certs/STAR.mobon.net.crt /etc/pki/ca-trust/source/anchors/  
  $ update-ca-trust

* mac  
  $ security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /apps/docker-registry/certs/STAR.mobon.net.crt  

`restart docker`  
$ service docker restart
> $ systemctl restart docker.service

docker registry에 접속할 각 장비에도 같은 경로로 ca.crt파일 복사해야함


#### create docker private registry container (default registry port : 5000)
$ mkdir -p /apps/docker-registry/auth
$ docker run --entrypoint htpasswd registry -Bbn mobon P@ssw0rd > /apps/docker-registry/auth/htpasswd

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

#### Docker 사용

--insecure-registry 설정
도커는 기본적으로 TLS(https)를 사용하므로 사설 저장소에 https를 설정하거나 아니면 --insecure-registry 설정을 해주고 docker daemon 제구동해야 함



## 9. Appendix

#### reference site

* 나만의 private docker registry 구성하기 
https://novemberde.github.io/2017/04/09/Docker_Registry_0.html

* 도커 이미지 개인 저장소(Docker Private Registry) 구축
https://m.blog.naver.com/PostView.nhn?blogId=complusblog&logNo=221000797682&proxyReferer=https%3A%2F%2Fwww.google.com%2F

* 사내 docker 저장소(registry) 구축하기
http://www.kwangsiklee.com/2017/08/%EC%82%AC%EB%82%B4-docker-%EC%A0%80%EC%9E%A5%EC%86%8Cregistry-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/

* docker private registry
https://setyourmindpark.github.io/2018/02/06/docker/docker-4/
