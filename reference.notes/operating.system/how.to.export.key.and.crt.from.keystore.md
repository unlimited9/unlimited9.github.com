# how to export .key and .crt from keystore

#### to export regular keys you should use -importkeystore command
$ keytool -importkeystore -srckeystore STAR.mediacategory.com.keystore -destkeystore STAR.mediacategory.com.p12 -deststoretype PKCS12

#### convert certificates
$ openssl pkcs12 -in STAR.mediacategory.com.p12 -out STAR.mediacategory.com.pem  
$ openssl pkcs12 -in STAR.mediacategory.com.p12 -nodes -nocerts -out STAR.mediacategory.com.key

>#### export certificates
>$ keytool -exportcert -keystore [keystore] -alias [alias] -file [cert_file]

## 9. Appendix

#### reference site

* keystore로 받은 인증서에서 pem파일을 뽑아보자  
https://mystria.github.io/archivers/get-pem-from-keystore

* [SSL] Keystore에서 PEM 추출하기  
https://medium.com/@jinro4/ssl-keystore%EC%97%90%EC%84%9C-pem-%EC%B6%94%EC%B6%9C%ED%95%98%EA%B8%B0-20cf49102283

* 이미 발급된 인증서를 이용해 SSL 구성하기
https://confluence.curvc.com/pages/viewpage.action?pageId=20251958
