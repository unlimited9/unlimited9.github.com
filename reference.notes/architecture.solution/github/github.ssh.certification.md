# GitHub with PKI/TLS Certificate

##### RSA Key
```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/my/home/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /my/home/.ssh/id_rsa.
Your public key has been saved in /my/home/.ssh/id_rsa.pub.
The key fingerprint is:
...
The key's randomart image is:
+---[RSA 2048]----+
...
+----[SHA256]-----+
$ cat /my/home/.ssh/id_rsa.pub
...
```
#### GitHub에 Deploy Key 등록
GitHub repository > Settings > Deploy Keys  
click `Add deploy key`

#### Clone project
ssh 인증을 사용하는 방법이기 때문에 clone을 받을 때 http 인증이 아닌 ssh 인증으로 받아야 한다  

#### git 인증 시간 늘리기
$ git config --global credential.helper 'cache --timeout=3600'

## Appendix

#### reference site

+ [GitHub] (공용 서버 등에서) 로그인 없이 GitHub 사용하기  
https://blog.leocat.kr/notes/2017/11/27/github-deploy-key
