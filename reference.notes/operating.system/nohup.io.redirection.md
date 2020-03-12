#### nohup
$ nohup command 1>/dev/null 2>&1 &

#### nohup
>SSH Connection이 끊어져도 실행중이던 작업이 종료되지 않고 백그라운드에서 계속 실행한다.

#### I/O Redirection
bash와 다른 shell에서는 I/O redirection을 제공한다.

#### I/O streams numbers
- 표준입력(stdin): 0  
- 표준출력(stdout): 1  
- 표준에러(stderr): 2  
> /dev/null : 출력이 파기되어, 아무것도 출력이 되지 않음. (쓰레기통으로 표현됨)

#### 1>/dev/null 이란?  
화면에 출력되는 표준출력 쓰레기통(/dev/null)에 넣어버려 어떤 값도 화면에 출력되지 않는다.

#### 2>&1 이란? (stderr redirect to stdout)
2번 표준에러를 1번 표준출력으로 redirection 한다.  
(1번 표준출력을 /dev/null로 redirection을 했기 때문에, 2번 표준에러도 1번이 표준출력이 redirection하는 /dev/null로 redirection 한다. ( 2 => 1 => /dev/null )

#### & 이란?
&는 실행 작업을 백그라운드에서 실행한다.

## 9. Appendix

#### reference site

+ Job Control
http://web.mit.edu/gnu/doc/html/features_5.html
