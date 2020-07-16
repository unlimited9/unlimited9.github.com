# Angular tutorial

#### 앵귤러 튜토리얼 (Angular tutorial) - 1
1. Node.js 설치  
2. Node.js 설치 이후 타입스크립트 설치, cmd창에서 명령을 입력하자 : npm install -g typescript  
3. cmd창에서 명령을 입력하자 : npm i @angular/cli -g  
4. VS코드(비쥬얼 스튜디오 코드)를 설치하자.  
5. Angular 8 Snippets를 설치하자.  
6. 프로젝트를 만들 디렉토리를 정한 뒤 ng new 프로젝트명(여기서는 study로하였다)을 입력하자.  
7. 프로젝트명으로 디렉토리가 만들어지는데 해당 디렉토리로 이동해서 ng serve 명령어를 입력하자.  
8. src/app 디렉토리에서 app.component.ts 파일을 클릭하자  
9. 클릭한 파일에 constructor(){} 를 입력하고 console.log 등을 입력하여 보자.  

#### 앵귤러 튜토리얼 (Angular tutorial) - 2

1. app.component.ts는 자바스크립트의 document.ready 이후 실행할 *.js 파일 같은 개념이다.  
2. app.component.ts 내부 @Component안에 templateUrl은 화면을 구성하는 부분이다.  

#### 앵귤러 튜토리얼 (Angular tutorial) - 3
1. ts로 불리는 파일은 Javascript에서의 <script></script> 역할을 한다.  
2. ts로 불리는 파일은 각종 데이터, 변수 및 함수를 보관하고 있다.  
3. 화면을 구성하는 것은 html파일로 이루어진다.  
4. 특수한 기호를 활용하면 ts파일에서 정의한 내용을 가져올 수 있다.  -> 예: {{blabla}}, ngFor, NgIf...  

#### 앵귤러 튜토리얼 (Angular tutorial) - 4
데코레이터 = json형식 기반의 앵귤러 설정 방법  
앵귤러에서 화면구성 = 컴포넌트 : 컴포넌트 데코레이터를 통해서 사용  

1. 컴포넌트 파일은 화면을 구성하는 기초 파일 이다.  
2. 컴포넌트 파일에 쓰여진 코드가 읽혀지고 나서, 이후에 화면을 구성 한다.  
3. @Component는 앵귤러의 데코레이터 일종이다.  
4. 컴포넌트 파일을 구성 할 때는 @Component 데코레이터와 export class 이름{} 라는 부분이 필요하다.  

1. 데코레이터(@로 시작하는)는 앵귤러의 설정을 담당하여준다.  
2. 컴포넌트 데코레이터(@Component)는 화면구성을 담당하는 파일이다.  

#### 앵귤러 튜토리얼 (Angular tutorial) - 5

디렉티브 : ngIf, ngFor 등  
데코레이터 = 앵귤러가 정의한 기능을 파일에 부여



 







---

>#### Typescript  
>- ts파일은 자바스크립트 형태 파일이 아니며, 컴파일 되기 전 형태의 파일이다.  
>- ts파일은 java처럼 java 파일 형태이지 class 파일 형태가 아니다.  
>- ts파일은 반드시 tsc 명령어를 통해 js 파일로 컴파일 해야 된다 (예: tsc 대상파일.ts) -> js파일을 만들어줌  
>- 만들어진 js 파일은 html에 넣어서도 사용 가능하며, 일일이 하기 귀찮을 때는 node 대상.js를 입력하면 결과를 볼 수 있다.  

## appendix

#### reference.site

+ 앵귤러 튜토리얼 (Angular tutorial) - 1  
https://lts0606.tistory.com/112?category=711929  

+ TypeScript 시작  
https://lts0606.tistory.com/17  
