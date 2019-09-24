# SSH(Secure Shell Protocol)

## Private Key and Public Key

#### ssh-keygen
```
usage: ssh-keygen [-q] [-b bits] [-t dsa | ecdsa | ed25519 | rsa | rsa1]
                  [-N new_passphrase] [-C comment] [-f output_keyfile]
       ssh-keygen -p [-P old_passphrase] [-N new_passphrase] [-f keyfile]
       ssh-keygen -i [-m key_format] [-f input_keyfile]
       ssh-keygen -e [-m key_format] [-f input_keyfile]
       ssh-keygen -y [-f input_keyfile]
       ssh-keygen -c [-P passphrase] [-C comment] [-f keyfile]
       ssh-keygen -l [-v] [-E fingerprint_hash] [-f input_keyfile]
       ssh-keygen -B [-f input_keyfile]
       ssh-keygen -D pkcs11
       ssh-keygen -F hostname [-f known_hosts_file] [-l]
       ssh-keygen -H [-f known_hosts_file]
       ssh-keygen -R hostname [-f known_hosts_file]
       ssh-keygen -r hostname [-f input_keyfile] [-g]
       ssh-keygen -G output_file [-v] [-b bits] [-M memory] [-S start_point]
       ssh-keygen -T output_file -f input_file [-v] [-a rounds] [-J num_lines]
                  [-j start_line] [-K checkpt] [-W generator]
       ssh-keygen -s ca_key -I certificate_identity [-h] [-n principals]
                  [-O option] [-V validity_interval] [-z serial_number] file ...
       ssh-keygen -L [-f input_keyfile]
       ssh-keygen -A
       ssh-keygen -k -f krl_file [-u] [-s ca_public] [-z version_number]
                  file ...
       ssh-keygen -Q -f krl_file file ...
```

$ ssh-keygen -t rsa -b 4096 -C "unlimited9@gmail.com"
```
Generating public/private rsa key pair.
Enter file in which to save the key (/apps/.ssh/id_rsa): 
Created directory '/apps/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /apps/.ssh/id_rsa.
Your public key has been saved in /apps/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:a8o+...0/Y unlimited9@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
...
+----[SHA256]-----+
```
>-t : 암호화 타입
>-C : comment. github에서는 사용자의 로그인 ID 입력하도록 가이드한다.
>-b : 암호화 비트수. default : 2048. gitlab에서는 4096 사용

$ ls -al ~/.ssh  
id_rsa      : private key  
id_rsa.pub  : public key

#### generate SSH key
`github`  
Setting > SSH and GPG keys > SSH keys > New SSH key

`gitlab`  
User > Settings > SSH keys


#### set SSH file permission
$ chmod 700 ~/.ssh  
$ chmod 644 ~/.ssh/authorized_keys  
$ chmod 644 ~/.ssh/known_hosts  
$ chmod 644 ~/.ssh/config  
$ chmod 600 ~/.ssh/id_rsa  
$ chmod 644 ~/.ssh/id_rsa.pub

