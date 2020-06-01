## system specification

#### system information
$ dmesg > dmesg.txt

#### CPU info
$ cat /proc/cpuinfo

#### memory
$ cat /proc/meminfo  

$ free -m  

$ sudo dmidecode -t memory  
$ sudo dmidecode | grep DDR

#### disk
$ df -h

#### network : IP
$ ifconfig

#### hostname, kernel version
$ uname -a  

$ hostname

#### release info
$ cat /etc/issue.net

#### system architecture - bit
$ getconf LONG_BIT

#### disk type : 1 - hdd, 0 - ssd
$ lsblk -d -o name,rota
NAME ROTA
sda     1
sdb     0
