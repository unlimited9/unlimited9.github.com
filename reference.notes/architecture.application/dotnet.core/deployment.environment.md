# Nsuslab ggcore deployment 

## create and delopy docker container

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
ENTRYPOINT ["dotnet", "${ServiceName}.dll"]
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

## issue

## Excute

#### build
docker exec nsuslab.ggcore.build.3.1-bionic sh -c "~/build.GGCore.Frontend.sh ggpokeruk"  

#### deployment
docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-770 --build-arg DeployEnvironment=ggpokeruk --build-arg ServiceName=GGCore.Backoffice -f Dockerfile.ggcore.service .  
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8013:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_backoffice:build-770  

docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-770 --build-arg DeployEnvironment=ggpokeruk --build-arg ServiceName=GGCore.Web.Widget -f Dockerfile.ggcore.service .  
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8014:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_widget:build-770  

docker build -t 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-770 --build-arg DeployEnvironment=ggpokeruk --build-arg ServiceName=GGCore.Web.Promotion -f Dockerfile.ggcore.service.promotion .  
docker run -d -e ASPNETCORE_ENVIRONMENT=Development -e GGCORE_SERVICE_MODE=GGPOKERUK -p 8020:80 809599471177.dkr.ecr.ap-northeast-1.amazonaws.com/ggcore_ggpokeruk_promotion:build-770  

## Appendix

#### reference.site
