# Development Environment : window 10

## Local install 

#### Visual Studio

#### Git
1. git remote add origin https://github.com/pyloncloud/88agent.git  
1. git pull  
1. git checkout origin/develop_ggcore  

#### nodejs : https://nodejs.org
1. download and install  
1. npm install -g npm

#### Docker Install
1. Enable Virtualization
   - Task Manager\Performance\Virtualization : Enabled  
   - Control Panel\Programs\Turn Windows features on or off : Hyper-V  

   - Set BIOS : Intel Virtualization Technology : Enabled  

1. Install Docker Desktop for Windows  
   https://hub.docker.com/editions/community/docker-ce-desktop-windows/  

## Docker container install 

#### MSSQL
1. docker pull microsoft/mssql-server-linux
1. docker run --name nsuslab.ggcore.mssql -p 1401:1433 -d -v C:/ggcore/data/mssql:/var/opt/mssql -e ACCEPT_EULA=Y -e MSSQL_SA_PASSWORD=P@ssw0rd --restart=unless-stopped microsoft/mssql-server-linux:latest

>- docker stop nsuslab.ggcore.mssql
>- docker rm nsuslab.ggcore.mssql

#### Redis
1. docker pull redis
1. docker run --name nsuslab.ggcore.redis -d -p 6379:6379 --restart=unless-stopped redis

>- docker stop nsuslab.ggcore.redis
>- docker rm nsuslab.ggcore.redis

---

>#### .NET Docker Images
>docker hub : https://hub.docker.com/  
>
>`mcr.microsoft.com/dotnet/core/aspnet:3.1`  
>Linux 및 Windows에서 런타임 전용 및 ASP.NET Core 최적화가 포함된 ASP.NET Core(다중 아키텍처)  
>
>`mcr.microsoft.com/dotnet/core/sdk:3.1`  
>Linux 및 Windows에서 SDK가 포함된 .NET Core(다중 아키텍처)  










## Appendix

#### reference.site

+ [Windows 컨테이너] 1: Windows 컨테이너에 대한 이해  
https://tech.devsisters.com/posts/intro-windows-container

+ 공식 .NET Docker 이미지  
https://docs.microsoft.com/ko-kr/dotnet/architecture/microservices/net-core-net-framework-containers/official-net-docker-images

* Home > Download > .NET Core > 3.1 > .NET Core 3.1 SDK (v3.1.301) - Linux x64 Binaries
https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-3.1.301-linux-x64-binaries
