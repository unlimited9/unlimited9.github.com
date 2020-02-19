# IntelliJ IDEA + Thymeleaf : auto reload, live reload

#### add dependency : devtools 
$ vi build.gradle
```
...
implementation 'org.springframework.boot:spring-boot-devtools'
...
```

#### add property 
$ vi application.properties
```
...
spring.devtools.livereload.enabled=true
...
```

## 9. Appendix

#### reference site

+ [Python Tutorial] 파이썬 튜토리얼 목차
https://xeros.dev/112?category=728734

IntelliJ IDEA 에서 Thymeleaf 를 사용하고, auto-reload 와 live reload 를 이용한다(바뀌면 알아서 재시작되고 리프레시된다)
