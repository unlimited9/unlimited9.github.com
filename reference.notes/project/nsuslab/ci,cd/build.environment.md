# Nsuslab ggcore build 

## docker build application container

#### create container dotnet.core sdk : nsuslab.ggcore.build.3.1-bionic
docker run -d -it -v /home/ubuntu/repo/app:/app -v /home/ubuntu/repo/pgms:/pgms --name nsuslab.ggcore.build.3.1-bionic --restart=unless-stopped mcr.microsoft.com/dotnet/core/sdk:3.1-bionic  
>docker run -d -it -v C:/repo/app:/app -v C:/repo/pgms:/pgms --name nsuslab.ggcore.build.3.1-bionic --restart=unless-stopped mcr.microsoft.com/dotnet/core/sdk:3.1-bionic  
>docker run -d -it -v C:/repo/app:/app -v C:/repo/pgms:/pgms --name nsuslab.ggcore.build.3.1-bionic -p 22:22--restart=unless-stopped mcr.microsoft.com/dotnet/core/sdk:3.1-bionic  
>docker run -d -it --name nsuslab.ggcore.build.3.1-bionic -p 22:22 -p 80:8080 --restart=unless-stopped mcr.microsoft.com/dotnet/core/sdk:3.1-bionic  

>docker run -d -it --name nsuslab.ggcore.build -p 80:8080 --restart=unless-stopped mcr.microsoft.com/dotnet/core/sdk:3.1-bionic  

#### build container environment 
docker exec -it nsuslab.ggcore.build.3.1-bionic /bin/bash

<<<<<<<<<<<<<<<<<<<<  
`install packages`  
apt update -y  
apt-get install -y net-tools iproute2 vim  

`install NVM(Node Version Manager)`  
https://github.com/creationix/nvm  

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash  
source ~/.bashrc  

`install Node.js`  
https://nodejs.org/en/  

nvm install 10.16.0  
> nvm install 12.18.2  
> nvm install --lts  

$ nvm use 10.16.0

>export NODE_VERSION=12.18.2  
>>export NODE_VERSION=10.16.0  
>
>curl -SL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz" --output nodejs.tar.gz \  
>    && tar -xzf "nodejs.tar.gz" -C /usr/local --strip-components=1 \  
>    && rm nodejs.tar.gz \  
>    && ln -s /usr/local/bin/node /usr/local/bin/nodejs  

npm install -g npm  

`create directory`  
mkdir -p /apps/install /pgms /data /logs  
mkdir /pgms/ggcore  

`checkout source`  
cd  /pgms/ggcore  
git init  
git config credential.helper store  
git remote add origin https://github.com/pyloncloud/88agent.git  
git pull  
git checkout origin/develop_ggcore  

`create build script`  
vi /pgms/script/build.GGCore.Frontend.sh  
>vi ~/build.GGCore.Frontend.sh  

```
#!/bin/sh
STARTTIME=$(date +%s)

cd  /pgms/ggcore
git pull origin develop_ggcore

export DeployEnvironment=$1

cd /pgms/ggcore/GGCore.Backoffice/ClientApp
npm install
npm rebuild node-sass

cd /pgms/ggcore
dotnet restore "GGCore.Backoffice/GGCore.Backoffice.csproj" --configfile NuGet.config
dotnet restore "GGCore.Web.Widget/GGCore.Web.Widget.csproj" --configfile NuGet.config
dotnet restore "src/Services/GGCore.Web.Promotion/GGCore.Web.Promotion.csproj"

#dotnet restore "GGCore.Guardian/GGCore.Guardian.csproj"

cd /pgms/ggcore/GGCore.Backoffice
dotnet build "GGCore.Backoffice.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Backoffice
if [ $? -ne 0 ] ; then exit 1 ; fi
dotnet publish "GGCore.Backoffice.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Backoffice
if [ $? -ne 0 ] ; then exit 1 ; fi

cd /pgms/ggcore/GGCore.Web.Widget
dotnet build "GGCore.Web.Widget.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Widget
if [ $? -ne 0 ] ; then exit 1 ; fi
dotnet publish "GGCore.Web.Widget.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Widget
if [ $? -ne 0 ] ; then exit 1 ; fi
cp -R Scripts /app/$DeployEnvironment/GGCore.Web.Widget/Scripts
cp -R static /app/$DeployEnvironment/GGCore.Web.Widget/static

cd /pgms/ggcore/src/Services/GGCore.Web.Promotion
dotnet build "GGCore.Web.Promotion.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Promotion/build
if [ $? -ne 0 ] ; then exit 1 ; fi
dotnet publish "GGCore.Web.Promotion.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Promotion/publish
if [ $? -ne 0 ] ; then exit 1 ; fi

#cd /pgms/ggcore/GGCore.Guardian
#dotnet build "GGCore.Guardian.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Guardian/build
#dotnet publish "GGCore.Guardian.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Guardian/publish


ENDTIME=$(date +%s)
echo "It takes $(($ENDTIME - $STARTTIME)) seconds to complete this task..."
```

chmod +x /pgms/script/build.GGCore.Frontend.sh  
/pgms/script/build.GGCore.Frontend.sh ggpokeruk  

>chmod +x ~/build.GGCore.Frontend.sh  
>~/build.GGCore.Frontend.sh ggpokeruk  

<<<<<<<<<<<<<<<<<<<<

docker exec nsuslab.ggcore.build.3.1-bionic sh -c "~/build.GGCore.Frontend.sh ggpokeruk"  

>#### issue - fatal: could not read Username for 'https://github.com': No such device or address
>`sloved.`  
>git config credential.helper store
>
>`ref.`  
>_default : 15 minute_  
>git config credential.helper cache  
>
>_set 3600 sec_  
>git config credential.helper 'cache --timeout=3600'  
>
>_해당 git directory 이외 모든 git에 적용 : --global_  
>git config credential.helper store --global  

---

## docker image build : create and delopy docker container

#### create dotnet.core.aspnet docker container : nsuslab.ggcore.aspnet.base:3.1-bionic
vi Dockerfile.nsuslab.ggcore.aspnet.3.1-bionic  
```
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-bionic

RUN apt-get update \
    && apt-get install -y --no-install-recommends libgdiplus libc6-dev \
    && rm -rf /var/lib/apt/lists/*

ENV CORECLR_ENABLE_PROFILING=1 \
CORECLR_PROFILER="{36032161-FFC0-4B61-B559-F6C5D41BAE5A}" \
CORECLR_NEWRELIC_HOME="/app/newrelic" \
CORECLR_PROFILER_PATH="/app/newrelic/libNewRelicProfiler.so" \
NEWRELIC_HOME="/app/newrelic" \
NEW_RELIC_LICENSE_KEY="6e4918d755b8d41eeeaccc0328c9f417de46073f"
```
docker build -t nsuslab.ggcore.aspnet.base:3.1-bionic --pull -f Dockerfile.nsuslab.ggcore.aspnet.3.1-bionic .  
>docker rmi nsuslab.ggcore.aspnet.base:3.1-bionic  

#### create nsuslab.ggcore.service docker container 
vi Dockerfile.nsuslab.ggcore.service  
```
FROM nsuslab.ggcore.aspnet.base:3.1-bionic
WORKDIR /app
EXPOSE 80

ARG DeployEnvironment
ENV DeployEnvironment=$DeployEnvironment
ARG ServiceName
ENV ServiceName=$ServiceName

COPY ${DeployEnvironment}/${ServiceName} .
ENTRYPOINT ["sh", "-c", "dotnet $ServiceName.dll"]
```
`GGCore.Backoffice`  
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-770 --build-arg DeployEnvironment=ggpokeruk --build-arg ServiceName=GGCore.Backoffice -f Dockerfile.nsuslab.ggcore.service .  
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8013:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-770  

`GGCore.Web.Widget`  
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-770 --build-arg DeployEnvironment=ggpokeruk --build-arg ServiceName=GGCore.Web.Widget -f Dockerfile.nsuslab.ggcore.service .  
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8014:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-770  

#### create nsuslab.ggcore.service.agent docker container 
vi Dockerfile.nsuslab.ggcore.service.agent
```
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1.5-bionicdocker  AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

ARG DeployEnvironment
ENV DeployEnvironment=$DeployEnvironment
ARG ServiceName
ENV ServiceName=$ServiceName

COPY ${DeployEnvironment}/${ServiceName}/publish .
ENTRYPOINT ["sh", "-c", "dotnet $ServiceName.dll"]
```

`GGCore.Web.Promotion`  
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-770 --build-arg DeployEnvironment=ggpokeruk --build-arg ServiceName=GGCore.Web.Promotion -f Dockerfile.nsuslab.ggcore.service.agent .  
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8020:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-770  

---

## Excute

#### build script
vi ~/build.GGCore.Frontend.sh  
```
#!/bin/sh
STARTTIME=$(date +%s)

#docker exec nsuslab.ggcore.build.3.1-bionic sh -c "~/build.GGCore.Frontend.sh $1"

cd ~/repo

cat <<EOT > .dockerignore
**
!app/$1/GGCore.Backoffice/**
EOT
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:$2 --build-arg DeployEnvironment=$1 --build-arg ServiceName=GGCore.Backoffice -f docker/Dockerfile.nsuslab.ggcore.service .
docker push 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:$2
#docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8013:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:$2

cat <<EOT > .dockerignore
**
!app/$1/GGCore.Web.Widget/**
EOT
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:$2 --build-arg DeployEnvironment=$1 --build-arg ServiceName=GGCore.Web.Widget -f docker/Dockerfile.nsuslab.ggcore.service .
docker push 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:$2
#docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8014:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:$2

cat <<EOT > .dockerignore
**
!app/$1/GGCore.Web.Promotion/**
EOT
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:$2 --build-arg DeployEnvironment=$1 --build-arg ServiceName=GGCore.Web.Promotion -f docker/Dockerfile.nsuslab.ggcore.service.agent .
docker push 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:$2
#docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8020:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:$2

ENDTIME=$(date +%s)
echo "It takes $(($ENDTIME - $STARTTIME)) seconds to complete this task..."
```

build.GGCore.Frontend.sh ggpokeruk build-test  

## Appendix

#### reference.site

- 올바른 Dockerfile 작성을 위한 가이드라인  
https://swalloow.github.io/dockerfile-ignore/  

* Angular / ng 빌드  
https://angular.io/cli/build  


