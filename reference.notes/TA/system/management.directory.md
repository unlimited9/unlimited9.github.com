## creating base directory

#### 디렉토리 생성
$ mkdir -p /apps/install /pgms /data /logs  

>~~$ mkdir -p /apps~~  
$ mkdir -p /apps/install  
$ mkdir -p /pgms  
$ mkdir -p /data  
$ mkdir -p /logs

#### 소유권 변경
$ chown -R app:app /apps /pgms /data /logs
>$ chown -R app:app /apps  
$ chown -R app:app /pgms  
$ chown -R app:app /data  
$ chown -R app:app /logs
