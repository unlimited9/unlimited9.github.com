## How to use special permissions: the setuid, setgid and sticky bits

#### file permissions
<table>
	<thead>
		<tr>
			<th>파일종류</th>
			<th colspan="3">특수권한</th>
			<th colspan="3">소유자 접근권한</th>
			<th colspan="3">그룹 소유자 접근권한</th>
			<th colspan="3">기타 사용자 접근 권한</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td rowspan="2">-,d,c,b,s,l,p</td>
			<td>4</td><td>2</td><td>1</td>
			<td>4</td><td>2</td><td>1</td>
			<td>4</td><td>2</td><td>1</td>
			<td>4</td><td>2</td><td>1</td>
		</tr>
		<tr>
			<td>setuid</td><td>setgid</td><td>sticky bit</td>
			<td>r</td><td>w</td><td>x</td>
			<td>r</td><td>w</td><td>x</td>
			<td>r</td><td>w</td><td>x</td>
		</tr>
	</tbody>
</table>

## trouble-shooting

#### 자바(Java) 실행 파일에 setuid 설정하기
tom
/usr/local/java/bin/java: error while loading shared libraries: libjli.so: cannot open shared object file: No such file or directory  

>ELF specification은 SUID와 SGID 바이너리에 대해 $ORIGIN을 무시하도록 권장한다.  
.so 파일 로딩 시 DT_RPATH나 DT_RUNPATH를 참조하는데 이 경로 내에서 $ORIGIN이 사용되기 때문에 문제가 발생한다.  
SUID나 SGID가 설정된 바이너리에서 $ORIGIN이 expanding되지 않도록 하거나, unset하는 이유는 바이너리에 대한 hard link를 다른 위치에 생성시켜 로딩되는 .so 파일을 바꿔치기 할 수 있기 때문이다.  

`libjli.so의 위치를 확인`  
$ ls -l $(locate libjli.so)

>warning: locate: could not open database: /var/lib/slocate/slocate.db: No such file or directory  
warning: You need to run the 'updatedb' command (as root) to create the database.  
Please have a look at /etc/updatedb.conf to enable the daily cron job.  
>$ sudo updatedb

`추가 library 참조`  
기본적으로 참조되는 /usr/lib이나 /lib에 위치하고 있지 않다면, 다음과 같이 global하게 참조되도록 해줄 수 있다.  
$ vi /etc/ld.so.conf.d/java.conf  
> libjli.so를 포함하는 디렉토리의 절대 경로를 추가  

$ /sbin/ldconfig  

`의존하고 있는 공유 라이브러리 (shared library)를 확인`
$ ldd /usr/local/java/bin/java
```
  linux-vdso.so.1 =>  (0x00007ffd98367000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x0000003f32e00000)
	libjli.so => /usr/local/java/bin/../lib/amd64/jli/libjli.so (0x00007f682cee3000)
	libdl.so.2 => /lib64/libdl.so.2 (0x0000003f33200000)
	libc.so.6 => /lib64/libc.so.6 (0x0000003f32a00000)
	/lib64/ld-linux-x86-64.so.2 (0x0000003f32600000)
```
> libjli.so 외의 파일은 global하게 참조되는 위치이므로 문제가 되지 않는다.

## Appendix

#### reference site

+ [UNIX / Linux] 특수 권한(setuid, setgid, sticky bit)  
https://eunguru.tistory.com/115

- 자바 (Java) 실행 파일에 setuid 설정하기  
https://devday.tistory.com/entry/%EC%9E%90%EB%B0%94-Java-%EC%8B%A4%ED%96%89-%ED%8C%8C%EC%9D%BC%EC%97%90-setuid-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0

- Why setuid java programs don't work  
http://blog.tinola.com/?e=7
