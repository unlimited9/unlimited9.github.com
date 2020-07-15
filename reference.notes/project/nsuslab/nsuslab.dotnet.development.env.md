# Development Environment

## Docker install
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

## .NET Core SDK : version 3.1 install

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

## VSCode install

#### download : https://code.visualstudio.com/download
https://go.microsoft.com/fwlink/?LinkID=620884

## MS SQL Server install


## Appendix

#### reference.site

* Home > Download > .NET Core > 3.1 > .NET Core 3.1 SDK (v3.1.301) - Linux x64 Binaries
https://dotnet.microsoft.com/download/dotnet-core/thank-you/sdk-3.1.301-linux-x64-binaries
