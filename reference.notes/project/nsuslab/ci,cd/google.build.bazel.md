# Installing Bazel on Ubuntu

## Using the binary installer

#### Step 1: Install required packages
sudo apt install g++ unzip zip  
>sudo apt-get install openjdk-11-jdk  

#### Step 2: Run the installer
'download the Bazel binary installer named bazel-<version>-installer-linux-x86_64.sh from the Bazel releases page on GitHub.' : https://github.com/bazelbuild/bazel/releases  

cd /apps/install  
wget https://github.com/bazelbuild/bazel/releases/download/3.4.1/bazel-3.4.1-installer-linux-x86_64.sh  
chmod +x bazel-3.4.1-installer-linux-x86_64.sh  
./bazel-3.4.1-installer-linux-x86_64.sh --prefix=/apps/bazel/3.4.1  

vi ~/.bashrc  
```
...
export PATH="$PATH:/apps/bazel/3.4.1/bin"
```

source ~/.bashrc  

# Bazel and Angular

npm install -g @angular/cli  
> error sh: 1: node: Permission denied  
npm install --unsafe-perm -g @angular/cli  

#### create angular project
> ng new bazel-angular --collection=@angular/bazel  
ng new angular-bazel  
ng add @angular/bazel   
```
Installing packages for tooling via npm.
An unhandled exception occurred: npm WARN @angular/bazel@9.1.12 requires a peer of @angular/compiler-cli@9.1.12 but none is installed. You must install peer dependencies yourself.
npm WARN @angular/bazel@9.1.12 requires a peer of @bazel/typescript@>=1.0.0 but none is installed. You must install peer dependencies yourself.
npm WARN @angular/bazel@9.1.12 requires a peer of rollup-plugin-commonjs@>=9.0.0 but none is installed. You must install peer dependencies yourself.
npm WARN @angular/bazel@9.1.12 requires a peer of rollup-plugin-node-resolve@>=4.2.0 but none is installed. You must install peer dependencies yourself.
npm WARN @angular/bazel@9.1.12 requires a peer of rollup-plugin-sourcemaps@>=0.4.0 but none is installed. You must install peer dependencies yourself.
npm WARN tsickle@0.38.1 requires a peer of typescript@~3.8.2 but none is installed. You must install peer dependencies yourself.

npm ERR! code ENOENT
npm ERR! syscall rename
npm ERR! path /pgms/temp/GGCore.Backoffice/ClientApp/node_modules/ngx-bootstrap
npm ERR! dest /pgms/temp/GGCore.Backoffice/ClientApp/node_modules/.ngx-bootstrap.DELETE
npm ERR! errno -2
npm ERR! enoent ENOENT: no such file or directory, rena​
32
ng new bazel-angular  
33
ng add @angular/bazel  me '/pgms/temp/GGCore.Backoffice/ClientApp/node_modules/ngx-bootstrap' -> '/pgms/temp/GGCore.Backoffice/ClientApp/node_modules/.ngx-bootstrap.DELETE'
npm ERR! enoent This is related to npm not being able to find a file.
npm ERR! enoent 

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2020-07-22T07_55_56_070Z-debug.log
Package install failed, see above.
See "/tmp/ng-1VMXfF/angular-errors.log" for further details.
```
ng build --leaveBazelFilesOnDisk  

## Build.bazel

#### bazelbuild / rules_nodejs

mkdir rules_nodejs.git  
cd rules_nodejs.git/  

git init  
git remote add origin https://github.com/bazelbuild/rules_nodejs.git  
git pull  
git checkout origin/master  

npm install --global yarn  

yarn  

ng serve  


## appendix

#### reference.site

* Bazel  
https://www.bazel.build/  

* Installing Bazel on Ubuntu  
https://docs.bazel.build/versions/master/install-ubuntu.html  

* bazelbuild / rules_nodejs  
https://github.com/bazelbuild/rules_nodejs  

+ Angular + Bazel : 준비 중!  
https://dev.to/thisdotmedia/angular-bazel-getting-ready-4b0i
