# SSH Passwordless Login Using SSH Keygen

## 패스워드 없이 ssh key로 인증/접속
서버계정과 원계서버계정이 다른 경우 

#### 인증키 생성
$ ssh-keygen -f ~/.ssh/app -C app@172.20.0.103:7722
```
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/gitlab-runner/.ssh/app.
Your public key has been saved in /home/gitlab-runner/.ssh/app.pub.
The key fingerprint is:
SHA256:jAnsQqEkadJwLwPOhpzjJW2RNUCaG17l8gCqGWmApjg app@172.20.0.103:7722
The key's randomart image is:
+---[RSA 2048]----+
|**++++           |
|&BB=+ .          |
|%&+B+.           |
|E+O++. +         |
|o=. ..o S        |
|   .             |
|                 |
|                 |
|                 |
+----[SHA256]-----+
```

#### 파일 퍼미션 설정
$ chmod 700 ~/.ssh  
$ chmod 600 ~/.ssh/app  
$ chmod 644 ~/.ssh/app.pub

#### 공개키 확인
$ cat ~/.ssh/app.pub 
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzduY7XPOz6V0Zbde3KRVlB6gE7yEhdbsQOCgNbXwI4GkAq7fkhH55S8FUEZ4Npo6bIlcy/xI8hvvqcaEfsnuLxkS1SN6RrbJqU/IIaxfPtuV5J0jLgxgOR6JPEVoA7dxPwtrYQ/rhYfUMx2SwvYLnL3ixylTjsZB8vJSwghaLI52U+LHyvFU8qbvOZlzXmeZ1GY+6H0+97gzvNfOtHwIE8vKeX2RLySKKMQzqUC/Drke0eqzU7TQwQ6xJlC5BjISnQ5/NfNGASNSKwSdqY1bkou0Bzz2OXv6/d1OV8vFhS9W4p+rMaA8OiyM/9eeWr14Pm58yd6Yqt+4xcYrWns9/ app@172.20.0.103:7722
```

#### 원격 서버 인증키 설정
$ vi ~/.ssh/authorized_keys
```
# gitlab-runner@955632f9fca8 public rsa key
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzduY7XPOz6V0Zbde3KRVlB6gE7yEhdbsQOCgNbXwI4GkAq7fkhH55S8FUEZ4Npo6bIlcy/xI8hvvqcaEfsnuLxkS1SN6RrbJqU/IIaxfPtuV5J0jLgxgOR6JPEVoA7dxPwtrYQ/rhYfUMx2SwvYLnL3ixylTjsZB8vJSwghaLI52U+LHyvFU8qbvOZlzXmeZ1GY+6H0+97gzvNfOtHwIE8vKeX2RLySKKMQzqUC/Drke0eqzU7TQwQ6xJlC5BjISnQ5/NfNGASNSKwSdqY1bkou0Bzz2OXv6/d1OV8vFhS9W4p+rMaA8OiyM/9eeWr14Pm58yd6Yqt+4xcYrWns9/ app@172.20.0.103:7722
```

#### 원격 서버 파일 퍼미션 설정
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys

#### 서버 접속
$ ssh -i ~/.ssh/app -p 7722 app@172.20.0.103


> 서버계정과 원계서버계정이 같을 경우
>
> #### RSA 키 생성 
> $ ssh-keygen -t rsa
>```
>Generating public/private rsa key pair.  
>Enter file in which to save the key (/home/users/syncwas/.ssh/id_rsa):  
>Created directory '/home/users/syncwas/.ssh'.  
>Enter passphrase (empty for no passphrase):  
>Enter same passphrase again: mben  
>Your identification has been saved in /home/users/syncwas/.ssh/id_rsa.  
>Your public key has been saved in /home/users/syncwas/.ssh/id_rsa.pub.  
>The key fingerprint is:  
>5f:fd:0a:5b:a5:19:72:ce:c6:91:6e:09:fb:d7:b8:52 syncwas@Batch-02  
>The key's randomart image is:  
>+--[ RSA 2048]----+  
>| |  
>| |  
>| |  
>| . . |  
>| S + * .|  
>| . . XEO |  
>| . o.%.o|  
>| .B..o|  
>| ..+o |  
>+-----------------+
>```
>
>#### 파일 퍼미션 설정
>$ chmod 700 ~/.ssh
>$ chmod 600 ~/.ssh/id_rsa
>$ chmod 644 ~/.ssh/id_rsa.pub
>
>#### 공개키 확인
>$ cat ~/.ssh/id_rsa.pub 
>```
>ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAyzt8W7bjmLhherqDC1DQVh6ZHCMdG68Y8Kei7GxV7l1RI4S9zo+G4VFtQP9wdLinwXLTzfmSrct2G3M4rr6xvWfgstZfxu+sKjVRCAj/vnHvgYm8adNpaU+UhHK0tLwy0w/xvJQhA0PGQFJM5s2Ho98y8Gd1cbh9p7IUdHbkpYvjN35b5HegLyhRdNngwWoMwOFbAj05UJJdUtHeOQQtkhU3c+1ivOiHlfFn+Pt1BW0m0DEFXg90fhuwgrDeEb09b+xCDDQ9OBtrNvZr93Z/MtkMHIfE6vMRIXsD9cGDVDbu47Bn+ysmh6AI29SE0s8PtAngaRGr2DYaT9Y5z09WQQ== syncwas@Batch-02
>```
>
>#### 원격 서버 인증키 설정
>$ vi ~/.ssh/authorized_keys
>```
>...
># Batch-02 public rsa key  
>ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAyzt8W7bjmLhherqDC1DQVh6ZHCMdG68Y8Kei7GxV7l1RI4S9zo+G4VFtQP9wdLinwXLTzfmSrct2G3M4rr6xvWfgstZfxu+sKjVRCAj/vnHvgYm8adNpaU+UhHK0tLwy0w/xvJQhA0PGQFJM5s2Ho98y8Gd1cbh9p7IUdHbkpYvjN35b5HegLyhRdNngwWoMwOFbAj05UJJdUtHeOQQtkhU3c+1ivOiHlfFn+Pt1BW0m0DEFXg90fhuwgrDeEb09b+xCDDQ9OBtrNvZr93Z/MtkMHIfE6vMRIXsD9cGDVDbu47Bn+ysmh6AI29SE0s8PtAngaRGr2DYaT9Y5z09WQQ== syncwas@Batch-02
>```
>
>#### 원격 서버 파일 퍼미션 설정
>$ chmod 700 ~/.ssh
>$ chmod 600 ~/.ssh/authorized_keys
>
>#### 서버 접속
>$ ssh -p 7722 app@172.20.0.103


vvvvv체크vvvvvvvvvvvvvvvvvvvvvvvvvvv
#### ssh가 구동 확인
$ eval $(ssh-agent)

#### ssh 인증키 생성
$ sudo -Es  
$ ssh-keygen -t rsa

$ ls -al ${HOME}/.ssh/ 
id_rsa
id_rsa.pub

#### ssh 인증키 복제
$ ssh-copy {MasetrNode2}
$ ssh-copy {MasetrNode3}

#### ssh 접속
$ ssh -A 10.0.0.7

