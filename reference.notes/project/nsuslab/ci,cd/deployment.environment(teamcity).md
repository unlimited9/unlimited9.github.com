# Nsuslab ggcore deployment

## Dependency

#### Dependent Build
~~Build Configurations > Dependency :  On the Dependencies page, click Add new artifact dependency  
Depend on: GGCore-TEST / GGPOKERUK / Build  
Get artifacts from: Latest successful build  
Artifacts rules: NuGet.config~~

Build Configurations > Dependency :  On the Dependencies page, click Add new snapshot dependency  
Depend on: GGCore-TEST / GGPOKERUK / Build  

#### SSH Keys Management
SSH private key into a project  

`Uploading SSH Key to TeamCity Server`  
Project Settings > SSH Keys : On the SSH Keys page, click Upload SSH Key.  

#### teamcity SSH Exec
`Deployment Target`  
Target: 13.230.138.165  

`Deployment Credentials`  
Authentication method: Uploaded key  
Username: ubuntu  
Select key: aws-ggcore-dev.pem  

`SSH Commands`  
Commands:  
```
echo 'Logging in to AWS ECR...'
$(~/.local/bin/aws --profile ggcore ecr get-login --no-include-email --region ap-northeast-1)

echo 'Pulling image: build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%'
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker pull 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%

echo 'Stopping all containers...'
docker ps --format '{{.Image}} {{.ID}}' | grep "ggpokeruk_" | awk '{print $2}' | xargs -I {} docker stop {}
# docker stop $(docker ps -aq)

echo 'Removing all containers...'
docker rm $(docker ps -aq)

echo 'Starting web services'
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8013:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8014:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8020:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-%dep.GGCoreTest_Ggpokeruk_Build.build.number%
```

## issue

#### Exited (145) About a minute ago
Entrypoint가 오류가 나서 컨테이너가 생성이 안되면 아래와 같이 해당 이미지로 컨테이너를 생성하여 확인해 볼 수 있다.  
docker run -it --rm --entrypoint=/bin/bash -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 18013:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-6  


## Appendix

#### reference.site
