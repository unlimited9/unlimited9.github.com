# 서버 디스크 용량 확보를 위한 log/backup/temp 파일 삭제

## log/backup/temp 파일 삭제

#### Linux : Docker resource prune
crontab -e  
```
0 1 * * 2 /usr/bin/docker system prune -af
```

#### Windows : 일정 시간 이전 파일 삭제

C:\Windows\System32\forfiles.exe /P D:Backuplog /M *.log /D -15 /S /C "cmd /c del @file"  

C:\Windows\System32\forfiles.exe /P "C:\Users\Administrator\Desktop\backup" /D -90 /C "cmd /c rd /s /q @file"  
C:\Windows\System32\forfiles.exe /P "C:\Users\Administrator\Desktop\bulid" /C "cmd /c rd /s /q @file"  
C:\Windows\System32\forfiles.exe /P "C:\Users\Administrator\Desktop\TicketService\Logs" /D -90 /C "cmd /c del /s /q @file"  

## appendix

#### reference.site

* MS TechNet  
http://technet.microsoft.com/ko-kr/library/cc755872(WS.10).aspx  

+ [윈도우]일정 시간이 지난 폴던 압축 및 삭제 배치 파일만들기  
https://eunice513.tistory.com/134  

+ [윈도우] 날짜 지난 파일 자동 삭제  
https://blog.naver.com/vfxx/100205776346  

