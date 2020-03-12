# Groovy

## Installation/Setup/Configuration

### Directory

#### make directory
$ mkdir -p /apps/groovy-sdk

### Download

#### https://groovy.apache.org/download.html
$ cd /apps/install
$ curl -O https://dl.bintray.com/groovy/maven/apache-groovy-sdk-2.5.7.zip
>~~$ wget https://dl.bintray.com/groovy/maven/apache-groovy-sdk-2.5.7.zip~~

### Install

#### decompress tarball
$ tar -zxvf /apps/install/apache-groovy-sdk-2.5.7.zip

#### copy application directory
$ cp -R /apps/install/apache-groovy-sdk-2.5.7 /apps/groovy-sdk/2.5.7

>### with SDKMAN
>
>#### install sdkman
>$ curl -s get.sdkman.io | bash
>
>#### install groovy
>$ sdk install groovy
>
>#### list versions of groovy
>$ sdk ls groovy
>
>#### switch versions of groovy
>$ sdk use groovy 2.4.7
>
>#### current version of groovy
>$ groovy -version
>
>>#### posh-gvm
>>The initial name of SDKMAN was GVM and  posh-gvm  is a port of GVM for the Windows Powershell.
>>https://github.com/flofreud/posh-gvm
>>It works the same as SDKMAN, but instead of  `sdk`  , the command is  `gmv`.

## Getting Started



## 9. Appendix

### reference site

#### Apache Groovy
http://groovy-lang.org/

#### Getting started with groovy
https://riptutorial.com/en/groovy
https://riptutorial.com/ko/groovy

#### Interacting with a SQL database
http://docs.groovy-lang.org/latest/html/documentation/sql-userguide.html
