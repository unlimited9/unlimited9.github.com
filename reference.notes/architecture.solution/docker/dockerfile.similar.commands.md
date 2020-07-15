# dockerfile command

## ENV vs ARG

#### ENV
- 환경변수 지정
- ENV var or ENV var=value
- 표현 : $var or ${var} or ${var:-value} or ${var:+value}
- docker run 시에 --e 옵션을 활용하여 오버라이딩 할 수 있다.

#### ARG
- build 시점에만 사용되는 변수
- ARG var or ARG var=value
- 표현 : $var or ${var} or ${var:-value} or ${var:+value}
- docker build 시에 --build-arg 옵션을 활용하여 오버라이딩 할 수 있다.

## CMD vs ENTRYPOINT

#### CMD
- 컨테이너가 시작될 때 실행
- Dockerfile에서 한번만 사용 가능(마지막 CMD만 사용)
- CMD ["실행 파일", "매개 변수", "매개 변수 ..."]
- docker run [IMAGE] [COMMAND]에서 COMMAND를 넣으면 CMD가 무시

#### ENTRYPOINT
- CMD와 동일하게 컨테이너가 시작될 때 실행
- CMD와 같이 있으면 ENTRYPOINT는 실행 파일, CMD는 매개변수 역할을 함
- docker run --entrypoint="[COMMAND] [IMAGE]"를 사용하여 무시 가능

## ADD vs COPY
공통적으로 .dockerignore에 명시된 영역은 제외

#### ADD
- 파일 복사
- 압축 파일인 경우, 압축을 품
- 단, URL로 가져온 파일은 압축만 해제하고 풀지는 않음
- OS에 따라서 압축 해제 여부가 있음
- 파일은 소유 root:root과 기존 권한을 가짐
- URL은 소유 root:root과 600 권한을 가짐

#### COPY
- 파일 복사
- ADD와 달리 파일 그대로 가져옴
- 권한 그대로 설정

## appendix

#### reference.site

- [docker] 헷갈리는 Dockerfile 명령어 #90  
https://github.com/heowc/programming-study/issues/90  
