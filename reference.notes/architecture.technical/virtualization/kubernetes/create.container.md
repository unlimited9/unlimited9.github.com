
#### kubernetes pod : ggid-dev.build
vi gid-dev.build.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ggid-dev.build
  labels:
    service-name: ggid-dev.build
spec:
  containers:
  - name: ggid-dev-build-3-1-bionic
    image: mcr.microsoft.com/dotnet/core/sdk:3.1-bionic
    imagePullPolicy: Always
    command: ['sh', '-c', 'tail -f /dev/null']
```

`linkerd inject`  
cat ggid-dev.build.yaml | linkerd inject - | kubectl apply -f -

`container`  
kubectl exec -it ggid-dev.build /bin/bash

>`delete`  
>kubectl delete pod ggid-dev.build

---
### docker container: nsuslab.ggcore.build
>docker run -d -it --name nsuslab.ggcore.build -p 80:8080 --restart=unless-stopped mcr.microsoft.com/dotnet/core/sdk:3.1-bionic  
>  
>docker exec -it nsuslab.ggcore.build /bin/bash  
>
>>docker stop nsuslab.ggcore.build  
>  
>>docker rm nsuslab.ggcore.build  

---

`install packages`  
apt update -y  
apt-get install -y net-tools iproute2 vim  

`install sudo`  
apt-get install sudo

`create group/user : app/app`  
groupadd -g 3000 app  
useradd -d /apps -g 3000 -m -u 3000 -s /bin/bash app  
passwd app  
adduser app sudo

`create directory`  
mkdir -p /apps/install /pgms /data /logs
chown -R app.app /apps /pgms /data /logs

---

su - app

cd /apps/install

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
source ~/.bashrc

nvm install 10.16.0
npm install -g npm

mkdir /pgms/ggcore
cd /pgms/ggcore

git init
git config credential.helper store
git remote add origin https://github.com/pyloncloud/88agent.git
git pull
git checkout origin/develop_ggcore

sudo apt install g++ unzip zip
cd /apps/install
wget https://github.com/bazelbuild/bazel/releases/download/3.4.1/bazel-3.4.1-installer-linux-x86_64.sh
chmod +x bazel-3.4.1-installer-linux-x86_64.sh
./bazel-3.4.1-installer-linux-x86_64.sh --prefix=/apps/bazel/3.4.1
vi ~/.bashrc
source ~/.bashrc

npm install -g @angular/cli
npm install --global yarn

---

cd /pgms
cp -R ggcore/GGCore.Backoffice/ClientApp ggcore.backoffice
cd ggcore.backoffice

ng add @angular/bazel
```
Installing packages for tooling via npm.
Installed packages for tooling via npm.
Two or more projects are using identical roots. Unable to determine project using current working directory. Using default workspace project instead.
CREATE e2e/protractor.on-prepare.js (1183 bytes)
CREATE src/initialize_testbed.ts (435 bytes)
CREATE src/main.dev.ts (150 bytes)
CREATE src/main.prod.ts (214 bytes)
CREATE src/rollup.config.js (254 bytes)
CREATE src/rxjs_shims.js (1262 bytes)
CREATE angular.json.bak (17211 bytes)
UPDATE package.json (4832 bytes)
UPDATE angular.json (2327 bytes)
UPDATE .gitignore (527 bytes)
✔ Packages installed successfully.
```
ng build --leaveBazelFilesOnDisk
```
Error: Cannot find module 'yargs'
```
npm install protractor-retry

ng build


rollup-plugin-sourcemaps
@coreui/coreui
@popperjs/core

vi .bazelrc
vi WORKSPACE
vi BUILD.bazel
vi src/BUILD.bazel

npm i @angular/core
npm i @bazel/bazelisk
yarn

ng build
bazel build //src:prodapp
```
could not find peer dependency 'rollup-plugin-sourcemaps' of '@angular/bazel'
```
npm i rollup-plugin-sourcemaps

ng build
```
could not find peer dependency '@coreui/coreui' of '@coreui/angular'
```
npm i @coreui/coreui

```
could not find peer dependency '@popperjs/core' of '@coreui/coreui'
```
npm i @popperjs/core

ng build
```
ERROR: cannot load '@npm_bazel_karma//:browser_repositories.bzl': no such file
```


