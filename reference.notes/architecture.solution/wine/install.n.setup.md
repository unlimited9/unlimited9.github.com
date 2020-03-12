## install

### Step 1 – Setup PPA
$ sudo dpkg --add-architecture i386
$ wget -qO - https://dl.winehq.org/wine-builds/winehq.key | sudo apt-key add -

#####  Ubuntu 18.10 
$ sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ cosmic main'
#####  Ubuntu 18.04 
$ sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ bionic main'
#####  Ubuntu 16.04 
$ sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ xenial main'

### Step 2 – Install Wine on Ubuntu
$ sudo apt-get update
$ sudo apt-get install --install-recommends winehq-stable

$ sudo apt-get install aptitude
$ sudo aptitude install winehq-stable

### Step 3 – Check Wine Version
$ wine --version
```
wine-4.0
```

### Use Wine
$ wine notepad.exe



#### Wine
$ sudo apt-get install wine-stable
$ WINEARCH=win32 WINEPREFIX=~/.wine wine wineboot

#### Winetrick
$ sudo apt-get install winetricks
> #### Winetrick manual install / update
>$ wget  https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
>$ chmod +x winetricks
>$ sudo cp winetricks /usr/local/bin

> 
>$ sudo apt-get install cabextract
>$  https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetrickswget
>$ chmod 777 winetricks

#### Playonlinux
>$ sudo apt-get install playonlinux

$ sudo dpkg -i 패키지.deb

## remove

#### Wine
$ sudo apt-get --purge remove wine*
> $ sudo apt-get purge wine-stable
> $ sudo apt-get purge winetricks

$ rm -rf ~/.wine
$ rm -rf ~/.local/share/applications/wine

#### Playonlinux
>$ sudo apt-get remove playonlinux


#### How to Install Wine 4.0 on Ubuntu 18.04 & 16.04 LTS
https://tecadmin.net/install-wine-on-ubuntu/

#### 우분투 18.04 에서 카카오톡 설치하기(Install kakaotalk on ubuntu 18.04)
https://www.hahwul.com/2018/08/install-kakaotalk-on-ubuntu-18.04.html

#### Winetricks
https://wiki.winehq.org/Winetricks

#### PlayOnLinux
https://www.playonlinux.com/en/
https://www.playonlinux.com/en/download.html

#### Install Wine, Winetricks and PlayOnLinux on Ubuntu 18.04
https://www.configserverfirewall.com/ubuntu-linux/install-wine-ubuntu-18/

#### 우분투 16.04 데스크탑 버전에서 playonlinux을 이용한 윈도우 프로그램 설치 방법
https://idchowto.com/?p=29028

#### 대단한 Wine 프론트엔드, PlayonLinux
http://opensea.egloos.com/5133570

#### 리눅스에서 윈도용 게임하기. Wine ,wine-staging-CSMT, wine-d3d9
https://moordev.tistory.com/129
