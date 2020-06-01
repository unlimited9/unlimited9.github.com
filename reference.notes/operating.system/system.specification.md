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
```
NAME ROTA
sda     1
sdb     0
```
--

1. CPU 정보 확인  

#명령어  
cat /proc/cpuinfo  
cat /proc/cpuinfo > cpuinfo.txt                     /* cpuinfo.txt 파일에 cpuinfo 정보를 저장 */  
cat /proc/cpuinfo | grep "model name" | head -1     /* cpu "model name" 중 맨위 한줄만 출력 */  
cat /proc/cpuinfo | grep "physical id"              /* cpu 갯수 확인 (0이면 1개 0과 1이 나오면 2개 0,1,2,3 이 나오면 cpu는 4개 */  


2. 메모리 정보 확인  

top                                 /* 메모리 사용현황 */
cat /proc/meminfo                   /* 메모리 용량 */
cat /proc/meminfo > meminfo.txt     /* meminfo.txt 파일에 meminfo 정보를 저장 */
dmidecode -t 17                     /* 메모리 상세정보(Ex 메모리 타입,슬롯,사이즈 등 */



3. 디스크 정보 확인

cat /proc/scsi/scs         /* scsi */
cat /proc/ide/hda/model    /* hda */
cat /proc/mdstat           /* linux Software Raid */
 
smartctl -a /dev/sd*    /* 디스크의 모든 SMART 정보 출력 */
smartctl -i /dev/sd*    /* 디스크의 신원 정보 출력 */
 
fdisk -l     /* 연결돼있는 디스크 리스트 정보 출력 */
df -h        /* 파티션 및 용량 정보 */
du -sh       /* 현재 폴더 데이터 사용량 */



4. PCI 정보 확인

lspci			/* 모든 PCI 정보 출력 */
lspci | grep RAID	/* RAID 모델명 확인 */
lspci | grep Ethernet	/* 랜카드 모델명 확인 */
lspci | grep VGA	         /* 그래픽 모델명 확인 */



5. 그 외 명령어

dmidecode -t TYPE   /* TYPE - bios,system,baseboard(메인보드),chassis,processor,memory,slot 등 다양한 정보 출력 가능 */
mount	/* 마운트된 파일시스템 정보 출력 */
lsusb	/* usb 정보 출력 */
lsscsi	/* scsi 목록 출력 */
hwinfo	/* 각종 모든 하드웨어 정보 출력 (hwinfo 패키지 설치 필요함) */


