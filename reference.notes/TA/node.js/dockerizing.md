
# dockerize 

>Created 요일 20 월 2017  
how to get a Node.js application into a Docker container.

## 1. Install Docker

[Install and setup Docker](/reference.notes/TA/cloud/docker/install.n.setup.md)

## 2. Dockerizing a Node.js web app

#### A. Create the Node.js app
$ vi package.json
```json
{
  "name": "docker.web.app",
  "version": "0.1.0",
  "description": "Node.js on Docker",
  "author": "Scott Lee <unlimited9@gmail.com>",
  "main": "docker.node.main",
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```

#### B. Creating a Dockerfile
$ vi Dockerfile
```
FROM node:10

# 앱 디렉터리 생성
WORKDIR /usr/src/app

# 앱 의존성 설치
# 가능한 경우(npm@5+) package.json과 package-lock.json을 모두 복사하기 위해
# 와일드카드를 사용
COPY package*.json ./

RUN npm install
# 프로덕션을 위한 코드를 빌드하는 경우
# RUN npm ci --only=production

# 앱 소스 추가
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]

```

#### C. Create a .dockerignore file

#### D. Building Docker image

#### E. Run the image


#### C. download

#### D. install


## 9. Appendix

#### reference site

* Node.js 웹 앱의 도커라이징
https://nodejs.org/ko/docs/guides/nodejs-docker-webapp/

* Node : Docker Official Images
https://hub.docker.com/_/node/

* npm-package.json : Specifics of npm's package.json handling
https://docs.npmjs.com/files/package.json.html
