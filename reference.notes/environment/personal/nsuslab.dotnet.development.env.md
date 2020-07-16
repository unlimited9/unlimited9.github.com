# Development Environment

## install Docker  
curl -fsSL https://get.docker.com/ | sudo sh  
sudo usermod -aG docker $USER  

sudo vi /etc/docker/daemon.json  
```
{
	"graph": "/data/docker"
}
```
mv /var/lib/docker /data/docker  
sudo systemctl restart docker  

## install .NET Core SDK : version 3.1

#### download
https://download.visualstudio.microsoft.com/download/pr/8db2b522-7fa2-4903-97ec-d6d04d297a01/f467006b9098c2de256e40d2e2f36fea/dotnet-sdk-3.1.301-linux-x64.tar.gz

#### decompress tarball
mkdir -p /repository/20.utility/microsoft/dotnet.core/3.1 && tar zxf dotnet-sdk-3.1.301-linux-x64.tar.gz -C /repository/20.utility/microsoft/dotnet.core/3.1

#### set environment variables
sudo vi /etc/profile.d/dotnet.sdk.sh
```
export DOTNET_ROOT=/repository/20.utility/microsoft/dotnet.core/3.1
export PATH=$PATH:/repository/20.utility/microsoft/dotnet.core/3.1
```

## install Angular
1. [install node.js](../../architecture.solution/node.js/install.n.setup.md)
1. typescript  
   npm install -g typescript  
1. angular  
   npm i @angular/cli -g  

## install VSCode  

#### download : https://code.visualstudio.com/download
https://go.microsoft.com/fwlink/?LinkID=620884

#### install extention(module)
1. Korean Language Pack for Visual Studio Code  
1. Angular Snippets (Version 9)


## Appendix

#### reference.site

* Home > Download > .NET Core > 3.1 > .NET Core 3.1 SDK (v3.1.301) - Linux x64 Binaries
https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-3.1.301-linux-x64-binaries

* Introduction to the Angular Docs  
https://angular.io/docs  

+ 앵귤러 튜토리얼 (Angular tutorial) - 1  
https://lts0606.tistory.com/112?category=711929  


