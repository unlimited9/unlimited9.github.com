
# dockerize 

>Created 요일 20 월 2017  
how to get a Node.js application into a Docker container.

## 1. Install Docker

[Install and setup Docker](../cloud/docker/install.n.setup.md)

## 2. Dockerizing a Node.js web app

#### A. Add Node.js application
$ vi server.js
```
/*
var os = require('os');
var http = require('http');

var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end("Hello World! I'm "+os.hostname());

  //log
  console.log("["+Date(Date.now()).toLocaleString()+"] "+os.hostname());
}

var www = http.createServer(handleRequest);
www.listen(8080);
*/

'use strict';

const express = require('express');

// 상수
const PORT = 8080;
const HOST = '0.0.0.0';

// 앱
const app = express();
app.get('/', (req, res) => {
  res.send('Hello world\n');
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);

```

#### B. Create the Node.js app
$ vi package.json
```json
{
  "name": "mobon.tracker.srv",
  "version": "0.1.0",
  "description": "Node.js on Docker",
  "author": "Scott Lee <unlimited9@gmail.com>",
  "main": "mobon.tracker.srv",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```

#### C. Creating a Dockerfile
$ vi Dockerfile
```
FROM node:10

# Craete application directory
WORKDIR /pgms/node.js/mobon.tracker.srv/

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
