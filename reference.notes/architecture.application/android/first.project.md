

#### Error while waiting for device: Could not start AVD
`/dev/kvm device: permission denied`  
Grant current user access to /dev/kvm  

`install kvm`  
$ sudo apt install qemu-kvm  

``  
$ ls -al /dev/kvm  
```
crw-rw---- 1 root kvm 10, 232  6월 22 11:38 /dev/kvm
```

`get group members`  
$ grep kvm /etc/group  
```
kvm:x:130:
```

`add group member`  
$ sudo adduser $USER kvm

>`set file permission`  
>$ sudo chown $USER /dev/kvm  
