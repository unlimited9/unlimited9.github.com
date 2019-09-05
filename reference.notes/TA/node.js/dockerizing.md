
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

# Craete application directory
WORKDIR /pgms/node.js/mobon.tracker/

# Install application dependency
COPY package*.json ./

RUN npm install
# Build package level : production
#RUN npm ci --only=production

# Add application source
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
