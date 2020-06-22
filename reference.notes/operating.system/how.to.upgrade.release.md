## How to Upgrade Ubuntu 18.04 LTS to Ubuntu 20.04 LTS

$ sudo do-release-upgrade  
> $ sudo do-release-upgrade -d  

#### Ubuntu Version
$ cat /etc/lsb-release

#### Step 1) Apply all updates of installed packages
$ sudo apt update  
$ sudo apt upgrade -y  

$ sudo reboot  

#### Step 2) Remove unused Kernels and install ‘update-manager-core’
$ sudo apt --purge autoremove  

$ sudo apt install update-manager-core -y  

#### Step 3) Start Upgrade Process
$ sudo do-release-upgrade  

`There is no development version of an LTS available`  
$ sudo do-release-upgrade -d  

#### Step 4) Verify Upgrade
$ cat /etc/lsb-release  


## Appendix

#### reference site

+ How to Upgrade Ubuntu 18.04 LTS to Ubuntu 20.04 LTS  
https://www.linuxtechi.com/upgrade-ubuntu-18-04-lts-to-ubuntu-20-04-lts/  







