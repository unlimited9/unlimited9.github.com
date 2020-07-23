# Node.js

Node.js는 V8이라는 구글에서 개발한 고성능 자바스크립트 엔진으로 빌드된 서버 사이드 개발용 소프트웨어 플랫폼입니다.

## 1. Installation(basic)

#### NVM(Node Version Manager) : https://github.com/creationix/nvm
https://github.com/nvm-sh/nvm  

`Install script`  
To install or update nvm, you can use the install script using cURL
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash  

or Wget:  
$ wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash  

설치가 되면 ~/.bashrc, ~/.bash_profile, ~/.zshrc, ~/.profile 등의 프로파일에 nvm.sh이 실행되도록 다음 스크립트가 추가됩니다.  
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

$ source ~/.bashrc  

`Verify installation`  
$ command -v nvm

#### Node.js : https://nodejs.org/en/
`Node 버전 설치`  
$ nvm install <version> # ex> nvm install 8.11.4  

`설치된 Node 버전 목록 확인`  
$ nvm ls  

`사용할 Node 설정`  
$ nvm use <version> # ex> nvm use 8.9.4  
$ nvm use <alias> # ex> nvm use default  

`사용할 alias 설정`  
$ nvm alias <alias> <version> # ex> nvm alias test-v 8.9.4  

`Node 삭제`  
$ nvm uninstall <version> # ex> nvm uninstall 8.9.4  

`Verify installation`  
$ command -v nvm  

## Development Environment (eclipse)

#### Eclipse Install : http://www.eclipse.org/downloads/
`Package Install`  
Eclipse IDE for Java EE Developers (recommended)  
>or Eclipse IDE for JavaScript and Web Developers  

#### Create Project (JavaScript)

`Create Project`  
File > New > Project > JavaScript > JavaScript Project  
Project Name : prj.surface  
`Encoding setup`  
Window > Preferences > General > Workspace > Text file encoding : UTF-8  
`NPM Initialize`  
File > New > Other... > JavaScript > npm Init  
`Node.js Module Install`  
1. open terminal  
  - add/create eclipse external script : run a program  
    Run > External Tools > External Tools Configurations  
      Project Name : terminalOpen  
      Location : /usr/bin/xfce4-terminal  
      Working Directory : ${resource_loc}  
2. Express Module 
# Express : 웹 요청을 처리
# Realm : 데이터베이스 활용
# EJS(Embedded JavaScript) : 템플릿 처리
# body-parser : 폼으로 전달된 쿼리를 처리

# Morgan : 메시지 콘솔 표시
# Compression : 페이지 압축 전송
# Session : 세션 처리
# Cookie-parser : 쿠키 사용
# Method-override : REST API에서 PUT과 DELETE 메소드를 사용
# Cors : 크로스오리진(다른 도메인 간의 AJAX 요청) 가능
# Multer : 파일업로드

$ npm install --save express
$ npm install --save realm
$ npm install --save ejs
$ npm install --save body-parser

# “–save” option : 설치된 모듈을 package.json에 기록
eclipse project refresh
Dependent Packages/Modules Install with package.json
$ npm install
Create a new JavaScript file
File > New > JavaScript Source File
   File Name : index.js
4. Environment Extensions (optional)

>> 

node.js plugin : Express
Global Install
$ npm install -g express
$ npm install -gd express-generator
Eclipse plugin
Nodeclipse Plugin Install
-. Help > Eclipse Marketplace... > "node" Search
-. Nodeclipse Install
Nodeclipse Plugin Setup
-. Window > Preferences > Nodeclipse
   Node Path: ~/.nvm/versions/node/v8.11.4/bin/node
   Express Path: ~/.nvm/versions/node/v8.11.4/bin/express
5. Project Extensions (optional)

>> 
Create Node.js Express Project
-. File > New > Node Project
   Project Name : mobon.ad.surface
   Template Engine : select ejs
   Stylesheet Engine : select stylus
Execute
-. Project Explorer > mobon.ad.surface > (right click) Run As > Node.js Application
   Main file : ${workspace_loc:/mobon.ad.surface/bin/www}
   Apply > Run
   Error: Cannot find module 'http-errors' ...
Dependent Packages/Modules Install with package.json
-. Project Explorer > mobon.ad.surface > (right click) StartExplorer > Start Shell Here
   npm install

9. Troubleshooting

>> ISSUE

/bin/bash: 줄 1: npm: 명령어를 찾을 수 없음.
cause
eclipse가 non-login shell로 실행되면서 /etc/profile과 /etc/bashrc만 실행되고 login(interactive) shell에서 사용하는 ~/.bashrc, ~/.profile, ~/.bash_profile이 실행되지 않는다.
solution
non-login shell에서 사용하는 /etc/profile과 /etc/bashrc 스크립트에 Path 추가
eclipse를 login shell로 기동 : ex) bash -ic "eclipse"
(nvm 이하 node.js가 ~/.nvm에 설치되므로 2번으로 해결)
X. Appendix

>> reference site

## Express 코어 및 미들웨어 시스템에 대한 변경
Express 4는 더 이상 Connect에 종속되지 않으며, express.static 함수를 제외한 모든 기본 제공 미들웨어가 Express 4 코어에서 제거되었습니다. 따라서 Express는 이제 독립적인 라우팅 및 미들웨어 웹 프레임워크가 되었으며, Express 버전화 및 릴리스는 미들웨어 업데이트의 영향을 받지 않게 되었습니다.
기본 제공 미들웨어가 없으므로 사용자는 앱을 실행하는 데 필요한 모든 미들웨어를 명시적으로 추가해야 합니다. 이를 위해서는 다음 단계를 따르기만 하면 됩니다.
모듈 설치: npm install --save <module-name>
앱 내에서, 모듈 요청: require('module-name')
해당 모듈의 문서에 따라 모듈 사용: app.use( ... )

다음 표에는 Express 3의 미들웨어 및 그에 대응하는 Express 4의 미들웨어가 나열되어 있습니다.
Express 3	Express 4
express.bodyParser	body-parser + multer
express.compress	compression
express.cookieSession	cookie-session
express.cookieParser	cookie-parser
express.logger	morgan
express.session	express-session
express.favicon	serve-favicon
express.responseTime	response-time
express.errorHandler	errorhandler
express.methodOverride	method-override
express.timeout	connect-timeout
express.vhost	vhost
express.csrf	csurf
express.directory	serve-index
express.static	serve-static
Express 4 미들웨어의 전체 목록을 참조하십시오.

>> PM2 - Node.js 프로세스 관리 도구
pm2 명령어를 사용해야 하므로 npm을 이용해서 전역으로 설치한다.
$ npm install pm2 -g
$ pm2 version

$ pm2 start app.js
$ pm2 list
$ pm2 show app
$ pm2 delete app


>> reference site
Node.js 개발환경 구성
teraspoon.egloos.com/1779944
Realm Node.js와 Express로 블로그 만들기
https://academy.realm.io/kr/posts/realm-node-js-express-blog-tutorial/
Node.js 애플리케이션의 간단한 프로파일링
https://nodejs.org/ko/docs/guides/simple-profiling/
Express 4로의 이전
http://expressjs.com/ko/guide/migrating-4.html
Use EJS to Template Your Node Application
https://scotch.io/tutorials/use-ejs-to-template-your-node-application
vue : https://github.com/vuejs/vue
Vue.js is an MIT-licensed open source project. It's an independent project with its ongoing development made possible entirely thanks to the support by these awesome backers. If you'd like to join them, please consider:
Element : http://element.eleme.io
Element, a Vue 2.0 based component library for developers, designers and product managers
vue-element-admin : https://github.com/PanJiaChen/vue-element-admin
vue-element-admin is a front-end management background integration solution. It based on vue and use the UI Toolkit element.
익스프레스 템플릿(Jade, Pug), Express template
https://www.zerocho.com/category/NodeJS/post/578c64621e3613150037d3b3
노드 프로젝트의 설정 파일들
https://www.zerocho.com/category/NodeJS/post/5b934c8e6c974e001b710767
PM2 - Node.js 프로세스 관리 도구
https://blog.outsider.ne.kr/1197






