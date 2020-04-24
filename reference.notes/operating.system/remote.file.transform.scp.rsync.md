## Unix/Linux scp 와 rsync : 원격파일 전송

### scp

#### usage
```
$ scp [-12346BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file]
      [-l limit] [-o ssh_option] [-P port] [-S program]
      [[user@]host1:]file1 ... [[user@]host2:]file2
```
> $ scp -[옵션] [보낼파일] [받는서버 계정 아이디]@[받는서버 URL]:[받을 위치 절대 경로]

>#### 옵션
>-r : recursive 하위 폴더 포함 모두 복사
-p : preserve 권한및 속성 유지
-C : compression 압축

$ scp -i ~/.ssh/app -P 7722 mobon.ad.tracker/mobon.tracker.application/build/libs/*.war app@172.20.0.103:/pgms/tomcat/wars/mobon.tracker

### rsync

#### usage
```
$ rsync [OPTION]... SRC [SRC]... DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
  or   rsync [OPTION]... SRC [SRC]... rsync://[USER@]HOST[:PORT]/DEST
  or   rsync [OPTION]... [USER@]HOST:SRC [DEST]
  or   rsync [OPTION]... [USER@]HOST::SRC [DEST]
  or   rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]

The ':' usages connect via remote shell, while '::' & 'rsync://' usages connect
to an rsync daemon, and require SRC or DEST to start with a module name.
```
> $ rsync -[옵션] [보낼파일] [받을위치]

>#### 옵션
>-v : 진행상황을 상세히 보여줌
-r : 지정한 디렉토리의 하위 디렉토리까지 재귀적으로 실행
-p : 버전속성 보존
-z : 데이터압축 전송
-u : 추가된 파일만 전송
-b : 낡은 파일은 ~가 붙음
-u : 새로운 파일을 덮어쓰지 않음
-e : ssh(rsh) 전송암호화

#### receive
$ rsync -a -e 'ssh -p 7722' syncwas@10.251.0.3:/home/dreamsearch/logs/gather/ 

#### send
$ rsync -az /home/users/syncwas/datas/*.clk rsync://14.0.92.22:/dmp/

