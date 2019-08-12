### J2EE 개발을 위한 통합개발환경 도구 Eclipse IDE 설치


1. Download : http://www.eclipse.org/downloads/eclipse-packages/
 * Eclipse IDE for Java Developers
    ```
    웹개발을 한다면 WTP(Web Tools Platform)를 설치한다.

    Step 1: Go to  Help -> Install New Software  -> Work with: [Eclipse release site]
    Step 2: Click on checkbox - Web, XML, Java EE and OSGi Enterprise Development.
    Step 3: Install"
    
    WTP 전체를 체크하면 incubator까시 설치된다.
    플러그인을 선택적으로 설치하지 않을 거라면 아래 Eclipse IDE for Java EE Developers를 설치한다.
    ```
 * Eclipse IDE for Java EE Developers (Recommend)
2. Eclipse option (JVM Argument)
 * excute(command line)
    ```
    $ eclipse.exe -vmargs -Xverify:none -XX:+UseParallelGC -XX:PermSize=256M -XX:NewSize=256M -XX:MaxNewSize=256M -Xmx1024m -Xms1024m
    ```
  * configuration(eclipse.ini)
-vmargs
-Dosgi.requiredJavaVersion=1.6
-Xverify:none
-XX:+AggressiveOpts
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnabled
-XX:+UseParNewGC
-XX:PermSize=256M
-XX:MaxPermSize=256M
-XX:NewSize=256M
-XX:MaxNewSize=256M
-Xms1024m
-Xmx1024m
---
#  *Plugins*
** Eclipse Class Decompiler
** STS(Spring Tool Suite)
** Subclipse (SVN, subversion)
** optional)
*** Jboss Tools
*** Gradle IDE
*** Papyrus UML
