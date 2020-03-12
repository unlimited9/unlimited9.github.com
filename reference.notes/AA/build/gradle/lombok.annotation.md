### Lombok annotation : cannot find symbol

#### > error message
```
> Task :compileTestJava FAILED  
/home/gitlab-runner/builds/8801fd62/0/enliple/mobon/library/utils/mobon.ad.utils/src/test/java/com/mobon/util/DateUtilsTest.java:26: error: cannot find symbol  
log.info(date);  
^  
symbol: variable log  
location: class DateUtilsTest  
1 error  
  
FAILURE: Build failed with an exception.
```

#### > solution : add dependencies - annotationProcessor and library
$ vi build.gradle
```gradle
ext {
	...
	lombokVersion = "1.18.6"
	...
}
...
dependencies {
	...
	annotationProcessor "org.projectlombok:lombok:${lombokVersion}"
	compileOnly "org.projectlombok:lombok:${lombokVersion}"
	testAnnotationProcessor "org.projectlombok:lombok:${lombokVersion}"
	testCompileOnly "org.projectlombok:lombok:${lombokVersion}"
	...
}
```
