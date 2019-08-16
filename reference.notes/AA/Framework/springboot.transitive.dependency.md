## Springboot dependency management

### gradle

#### add spring dependency management plugin
$ vi build.gradle
```
...
apply plugin: 'io.spring.dependency-management'
...
```

#### before project dependencies
```
...
org.elasticsearch.client:elasticsearch-rest-high-level-client:6.4.3
org.mongodb:mongo-java-driver:3.8.2
...
```

#### overriding dependency versions
$ vi build.gradle
```
...
ext {
    set('elasticsearch.version', '7.1.1')
    set('mongodb.version', '3.10.2')
}
...
```

#### after project dependencies
```
...
org.elasticsearch.client:elasticsearch-rest-high-level-client:7.1.1
org.mongodb:mongo-java-driver:3.10.2
...
```

## 9. Appendix

### reference site

- Managing Transitive Dependencies
https://docs.gradle.org/current/userguide/managing_transitive_dependencies.html#sec:excluding_transitive_module_dependencies

- Overriding Dependency Versions with Spring Boot
https://spring.io/blog/2016/04/13/overriding-dependency-versions-with-spring-boot

