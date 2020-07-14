# Nsuslab ggcore build 

## docker build container

#### create container dotnet.core sdk : nsuslab.ggcore.build.3.1-bionic
docker run -d -it -v C:/ggcore/pgms/build:/app --name nsuslab.ggcore.build.3.1-bionic mcr.microsoft.com/dotnet/core/sdk:3.1-bionic

#### build container environment 
docker exec -it nsuslab.ggcore.build.3.1-bionic /bin/bash

<<<<<<<<<<<<<<<<<<<<
`install packages`  
apt update  
apt-get install -y net-tools iproute2 vim  

`install node.js`  
export NODE_VERSION=12.18.2  
curl -SL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz" --output nodejs.tar.gz \  
    && tar -xzf "nodejs.tar.gz" -C /usr/local --strip-components=1 \  
    && rm nodejs.tar.gz \  
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs  

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
vi ~/build.GGCore.Frontend.sh  
```
#!/bin/sh
STARTTIME=$(date +%s)

cd  /pgms/ggcore
git pull

export DeployEnvironment=$1

cd /pgms/ggcore/GGCore.Backoffice/ClientApp
npm install
npm rebuild node-sass

cd /pgms/ggcore
dotnet restore "GGCore.Backoffice/GGCore.Backoffice.csproj" --configfile NuGet.config
dotnet restore "GGCore.Web.Widget/GGCore.Web.Widget.csproj" --configfile NuGet.config
dotnet restore "src/Services/GGCore.Web.Promotion/GGCore.Web.Promotion.csproj"

dotnet restore "GGCore.Guardian/GGCore.Guardian.csproj"

cd /pgms/ggcore/GGCore.Backoffice
dotnet build "GGCore.Backoffice.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Backoffice
dotnet publish "GGCore.Backoffice.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Backoffice

cd /pgms/ggcore/GGCore.Web.Widget
dotnet build "GGCore.Web.Widget.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Widget
dotnet publish "GGCore.Web.Widget.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Widget
cp -R Scripts /app/$DeployEnvironment/GGCore.Web.Widget/Scripts
cp -R static /app/$DeployEnvironment/GGCore.Web.Widget/static

cd /pgms/ggcore/src/Services/GGCore.Web.Promotion
dotnet build "GGCore.Web.Promotion.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Promotion/build
dotnet publish "GGCore.Web.Promotion.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Web.Promotion/publish

cd /pgms/ggcore/src/Services/GGCore.Guardian
dotnet build "GGCore.Guardian.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Guardian/build
dotnet publish "GGCore.Guardian.csproj" -c Release -o /app/$DeployEnvironment/GGCore.Guardian/publish


ENDTIME=$(date +%s)
echo "It takes $(($ENDTIME - $STARTTIME)) seconds to complete this task..."
```
chmod +x ~/build.GGCore.Frontend.sh  
~/build.GGCore.Frontend.sh ggpokeruk  
<<<<<<<<<<<<<<<<<<<<

docker exec nsuslab.ggcore.build.3.1-bionic sh -c "~/build.GGCore.Frontend.sh ggpokeruk"  

#### build script
vi ~/build.GGCore.Frontend.sh  
```
#!/bin/sh
STARTTIME=$(date +%s)

cd ~/repo

docker exec nsuslab.ggcore.build.3.1-bionic sh -c "~/build.GGCore.Frontend.sh $1"

cat <<EOT > .dockerignore
**
!app/$1/GGCore.Backoffice/**
EOT
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:$2 --build-arg DeployEnvironment=$1 --build-arg ServiceName=GGCore.Backoffice -f docker/Dockerfile.nsuslab.ggcore.service .
#docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8013:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:$2

cat <<EOT > .dockerignore
**
!app/$1/GGCore.Web.Widget/**
EOT
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:$2 --build-arg DeployEnvironment=$1 --build-arg ServiceName=GGCore.Web.Widget -f docker/Dockerfile.nsuslab.ggcore.service .
#docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8014:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:$2

cat <<EOT > .dockerignore
**
!app/$1/GGCore.Web.Promotion/**
EOT
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:$2 --build-arg DeployEnvironment=$1 --build-arg ServiceName=GGCore.Web.Promotion -f docker/Dockerfile.nsuslab.ggcore.service.agent .
#docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8020:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:$2

ENDTIME=$(date +%s)
echo "It takes $(($ENDTIME - $STARTTIME)) seconds to complete this task..."
```

## issue

#### fatal: could not read Username for 'https://github.com': No such device or address
`sloved.`  
git config credential.helper store

`ref.`  
_default : 15 minute_  
git config credential.helper cache  

_set 3600 sec_  
git config credential.helper 'cache --timeout=3600'  

_해당 git directory 이외 모든 git에 적용 : --global_  
git config credential.helper store --global  

## Appendix

#### reference.site

- 올바른 Dockerfile 작성을 위한 가이드라인  
https://swalloow.github.io/dockerfile-ignore/  
