# JVM Memory Tuning

>Created 목요일 20 2월 2017
automating deployment, scaling, and management of containerized applications.


Xms1024m -Xmx1024m -XX:MaxPermSize=256m // Xms 와 Xmx 의 차이는 virtual 메모리임.
-XX:NewSize=100m -XX:MaxNewSize=100m -XX:SurvivorRatio=32 // young generation을 위한 옵션
-XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:-CMSParallelRemarkEnabled // gc 시간을 최소화
-XX:-DisableExplicitGC // System.gc() 를 disable 한다.
-XX:MaxTenuringThreshold=0 -XX:CMSInitiatingOccupancyFraction=60 // 성능을 위한 기타 추가 옵션
-verbose:gc -XX:+PrintGCTimeStamps -XX:+PrintGCDetails // gc print 옵션

[GC 설명]

GC 는 minor gc(young generation) 와 major gc (tenured generation) 로 구분된다.
minor gc 는 로그에 [GC] 로 표시되며 major gc 는 [Full GC]로 표시된다.


JVM 영역 


young generation 이 클수록 minor gc 는 적게 일어난다. 그러나 heap 사이즈의 영역에서 young generation 이 크다는 것은
상대적으로 tenured generation 작다는 것을 의미하므로 major gc 가 자주 일어나게 된다.
young generation 은 NewRatio 값으로 제어할 수 있다. 예를 들어 -XX:NewRatio=3 옵션은 young:tenured 가 1:3 의 비율을
유지한다는 것을 의미한다.
young generation 은 NewSize와 MaxNewSize로 지정할 수 있는데 두 합을 같이하면 Xms, Mmx 를 합한것을 사용하듯이
young generation 의 크기를 정할 수 있다.

young generation은 eden 영역과 first survivor, second survivor 영역으로 구성된다. 이상적인 minor gc는 eden과 first survivor에
저장되어 사용중인 객체가 second survivor로 복사되는 것이다. 그러나 second survivor 영역이 충분하다고 보장되지는 않는다.
그러므로 minor gc 를 성공하기 위해서 tenured generation에 오든 객체가 저장될 수 있는 충분한 메모리가 확보되어야 한다.
만약 tenured generation 영역과 eden + survivor 영역의 사용중인 객체 메모리가 같다면 더이상 minor gc 로 tenured generation 영역을
사용할 수 없으므로 major gc가 발생하게 된다.

SurvivorRatio 파라미터를 사용하면 survivor 영역을 지정할 수 있다. 예를 들어 -XX:SurvivorRatio=6 이라고 한다면
각 survivor 영역과 eden 의 영역이 1:6 이라는 의미이다. 그러므로 하나의 survivor 는 총 young generation 영역에서 1/8 이 된다.
(second 가 있으므로 1/7 이 아니라 1/8 이 된다.)


[4가지 type 의 collector]

1. default garbage collector

2. The throughput collector

-XX:+UseParallelGC 를 사용하는 방법

3. The concurrent low pause collector

CPU 가 2장이상 많으면 많을수록 적용하기 적합한 collector 이다.
concurrent low pause collector 는 운영중인 서버에 gc로 인한 pause 를 최소화 하기 위해서 사용되는 옵션이다.
tenured generation 도 함께 일어난다.
이 옵션을 적용할려면 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC 두가지 옵션을 주면 된다.
그러나 -XX:+UseParallelGC 옵션과 -XX:+UseParNewGC 옵션을 동시에 사용하면 안된다.

4. The incremental (sometimes called train) low pause collector

일부분은 tenured generation 이 일어나며 major gc 의 일시멈춤현상을 최소화 할 수 있다.
그러나 throughput 을 고려할 때 default collector 보다 느리다.
-Xincgc 옵션을 사용하면 되나 다른 옵션과 함께 사용하면 JVM이 어떻게 동작할지 예상할 수 없으므로
같이 사용하면 안된다.


[GC log format]

[GC [: -> , secs] -> , secs]

is an internal name for the collector used in the minor collection
is the occupancy of the young generation before the collection
is the occupancy of the young generation after the collection
is the pause time in seconds for the minor collection.
is the occupancy of the entire heap before the collection
is the occupancy of the entire heap after the collection
is the pause time for the entire garbage collection. This would include the time for a major collection is one was done.

[ GC log case 해석 ]

1. Young generation size is too small

[GC [DefNew: 4032K->64K(4032K), 0.0429742 secs] 9350K->7748K(32704K), 0.0431096 secs]

-> gc 가 잘 된것 같으나 9350K->7748K 로 줄어들었으니 총 줄어든 경우가 1602K 정도 밖에 되지 않는다.
전체 메모리중에서 40% 는 여전히 사용되고 있으니 이는 young generation 이 너무 작다는 이야기이다.

2. The young generation size is too large

[GC [DefNew: 16000K->16000K(16192K), 0.0000574 secs][Tenured: 2973K->2704K(16384K), 0.1012650 secs] 18973K->2704K(32576K), 0.1015066 secs]

-> 앞부분에 보면 16000K->16000K로 변함이 없다. 왜냐하면 이를 복사할 Tenured 가 상대적으로 적으니 변함이 없는 것이다.
이 경우는 young generation 줄어들려면 major gc 가 일어나야 줄어들 수 있다.
그래도 옵션에 의해서 Tenured 영역에 일부 gc가 일어나 18973K->2704K(32576K) 만큼 줄어든 결과를 보이고 있다.

3. Is the tenured generation too large or too small?

[GC [DefNew: 8128K->8128K(8128K), 0.0000558 secs][Tenured: 17746K->2309K(24576K), 0.1247669 secs] 25874K->2309K(32704K), 0.1250098 secs]

-> 총 메모리가 32M 일 경우 major gc 가 일어나 약 0.13초가 걸렸다.

[GC [DefNew: 8128K->8128K(8128K), 0.0000369 secs][Tenured: 50059K->5338K(57344K), 0.2218912 secs] 58187K->5338K(65472K), 0.2221317 secs]

-> 총 메모리가 64M 일 경우 major gc 가 일어나 약 0.23초가 걸렸다.

위의 둘을 비교해 볼 때 첫번째가 좋을 것 같으나 이는 gc 의 빈번도를 알아봐야 최종 결론을 낼 수 있다.

111.042: [GC 111.042: [DefNew: 8128K->8128K(8128K), 0.0000505 secs]111.042: [Tenured: 18154K->2311K(24576K), 0.1290354 secs] 26282K->2311K(32704K), 0.1293306 secs]
122.463: [GC 122.463: [DefNew: 8128K->8128K(8128K), 0.0000560 secs]122.463: [Tenured: 18630K->2366K(24576K), 0.1322560 secs] 26758K->2366K(32704K), 0.1325284 secs]
133.896: [GC 133.897: [DefNew: 8128K->8128K(8128K), 0.0000443 secs]133.897: [Tenured: 18240K->2573K(24576K), 0.1340199 secs] 26368K->2573K(32704K), 0.1343218 secs]

-> 첫번째는 11초마다 일어난 반면에

90.597: [GC 90.597: [DefNew: 8128K->8128K(8128K), 0.0000542 secs]90.597: [Tenured: 49841K->5141K(57344K), 0.2129882 secs] 57969K->5141K(65472K), 0.2133274 secs]
120.899: [GC 120.899: [DefNew: 8128K->8128K(8128K), 0.0000550 secs]120.899: [Tenured: 50384K->2430K(57344K), 0.2216590 secs] 58512K->2430K(65472K), 0.2219384 secs]
153.968: [GC 153.968: [DefNew: 8128K->8128K(8128K), 0.0000511 secs]153.968: [Tenured: 51164K->2309K(57344K), 0.2193906 secs] 59292K->2309K(65472K), 0.2196372 secs]

-> 두번째는 30초마다 일어났으므로 두번째 메모리 세팅이 훨씬 좋음을 알 수 있다.

[The concurrent low pause collector 측정방법]

* -verbose:gc 와 -XX:+PrintGCDetails 옵션을 적용한다.
* CMS-initial-mark : initial marking 상태에 대한 GC 통계를 보여준다.
* CMS-concurrent-mark : concurrent marking 상태에 대한 GC 통계를 보여준다.
* CMS-concurrent-sweep : concurrent sweeping 상태에 대한 통계를 보여준다.
* CMS-concurrent-preclean : concurrently 하게 작동할 수 있도록 결정된 작업의 통계
* CMS-remark : remarking 상태에 대한 통계
* CMS-concurrent-reset : concurrent 작업이 끝나고 다음 collection을 준비한다.

[The concurrent low pause collector 사용시 Parallel Minor Collection 옵션]

* -XX:+UseParNewGC : 다중 CPU에서 멀티쓰레드를 사용한 young generation collection을 사용하는 옵션.
* -XX:+CMSParallelRemarkEnabled : remark 시 일시 중단을 멈추는 옵션
그러나 이 옵션은 jvm 1.4.2 에서 버그로 판명되어 (-) 로 사용해야 한다.
default 가 (+) 이므로 -XX:-CMSParallelRemarkEnabled 명시적으로 사용함.
JVM 1.5.0 (tiger)에서는 해결되었음

[기타 옵션]

1. -XX:MaxTenuringThreshold=0

default는 31 이며 이 옵션은 young generation 시에 객체가 살아 있을 수 있는 age 를 나타낸다.
이 옵션이 0 가 되면 young generation 시에 older generation 에 남아있는 객체가 복사되는
시간을 줄일 수 있다.

2. -XX:CMSInitiatingOccupancyFraction=60

이 옵션의 값이 특정 지정한 값보다 작으면 gc 가 자주 일어나면 값이 크면 gc 의 효율이 떨어질 수 있다.
그러므로 해당 값을 잘 정하는 것이 좋으며 일반적으로 60 으로 세팅하여 로그를 보고 조절하면 된다.


[ 기타 사항]

1. jsp 와 같이 동적으로 generation 되는 어플리케이션은 permanent generation 이 성능저하의 한 요소가 될 수 있으므로
-XX:MaxPermSize=256m 와 같은 옵션을 사용하여 해당 메모리를 늘려줄 필요가 있다.

2. 프로그래머가 System.gc() 를 호출하면 major gc 가 일어나므로 성능의 영향을 줄 수 있다.
그러므로 -XX:+DisableExplicitGC 옵션을 사용하여 프로그래머가 gc 를 일으키지 않도록 제한해야 한다.
JVM 1.4.2 에서는 버그가 있어 -XX:-DisableExplicitGC 와 같이 (-) 로 사용해야 해당 기능을 적용할 수 있다.

3. Concurrent low pause collector 는 Parallel Collector 와 Concurrent mark-sweep (CMS) collector 의 기능을
모두 가지고 있다.


[example]
java -Xmx512m -Xms512m -XX:MaxNewSize=24m -XX:NewSize=24m -XX:SurvivorRatio=128 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:MaxTenuringThreshold=0 -XX:CMSInitiatingOccupancyFraction=60





## 9. Appendix

#### reference site

+ JVM 메모리 관련 설정  
https://epthffh.tistory.com/entry/JVM-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B4%80%EB%A0%A8-%EC%84%A4%EC%A0%95
