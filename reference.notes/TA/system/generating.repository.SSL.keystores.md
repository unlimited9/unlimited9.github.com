## Instructions for Generating Repository SSL Keystores

#### 인증서 변환(to keystore)
$ openssl pkcs12 -export -in STAR.mediacategory.com.pem -inkey STAR.mediacategory.com.key -out keystore.p12 -name "mediacategory.com"


## Appendix

#### reference site

- java keytool 사용법 - Keystore 생성, 키쌍 생성, 인증서 등록 및 관리  
https://www.lesstif.com/pages/viewpage.action?pageId=20775436

- java keytool SSL 사설 인증서  
https://mycup.tistory.com/250

- [Spring Boot #7] 스프링부트(Spring Boot) HTTPS 구축, HTTP2, 다중 커넥터 설정  
https://engkimbs.tistory.com/756

- 스프링 부트로 letsencrypt를 이용하여 무료 SSL 올리기  
http://www.kwangsiklee.com/2016/12/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8%EB%A1%9C-letsencrypt%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-%EB%AC%B4%EB%A3%8C-ssl-%EC%98%AC%EB%A6%AC%EA%B8%B0/

- [TIP] 스프링 웹서비스에서 SSL을 위한 인증서 설정 방법  
http://airpage.org/xe/tool_data/23809

- Spring Boot SSL 인증서 설치/적용 가이드   
https://www.securesign.kr/guides/Spring-Boot-SSL-Certificate-Install

- Spring Boot HTTPS 적용 하기  
https://cheese10yun.github.io/spring-https/

- Spring Boot에서 HTTPS를 적용하려면?  
https://elfinlas.github.io/2017/11/29/springboot-https/
