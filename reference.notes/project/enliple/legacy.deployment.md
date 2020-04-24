# mobon legacy deployment

## 광고주: advertiser

<details>
<summary>L4: loadbalancer</summary>
<div markdown="1" desc="admin / enliple20!#">

$ telnet 119.205.238.46 23  
```
# /info/slb/virt 45

# /stats/slb/virt 29

# /cfg/slb/real 32
# dis
# diff
# ena
# apply
# save
```

</div>
</details>

<details>
<summary>upload binary source</summary>
<div markdown="1">

$ rsync -az /home/dreamsearch/public_html/WEB-INF rsync://10.251.0.3:/home/dreamsearch/public_html/

</div>
</details>

<details>
<summary>application deployment</summary>
<div markdown="1">

$ ssh sjlee@10.251.0.32 -p 7722
```
$ su -

$ sh /root/shell/was_sync.sh

```

</div>
</details>

## 송출: media

<details>
<summary>L4: loadbalancer</summary>
<div markdown="1" desc="admin / enliple20!#">

$ telnet 119.205.238.30 23  
```
# /info/slb/virt 45

# /stats/slb/virt 29

# /cfg/slb/real 32
# dis
# diff
# ena
# apply
# save
```

</div>
</details>

<details>
<summary>application deployment</summary>
<div markdown="1">

$ ssh sjlee@10.251.0.4 -p 7722
```
$ su -

$ sh /root/shell/was_sync.sh

```

</div>
</details>

## 9. Appendix

#### reference site

+ 모비온 배포 프로세스
https://www.evernote.com/l/AjiBQhIHCPJNVoirWe6F4edgqmuQV1tyr5s/
