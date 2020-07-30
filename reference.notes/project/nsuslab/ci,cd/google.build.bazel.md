# Installing Bazel on Ubuntu

bazel is coordination tool!!!

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
ng build --leaveBazelFilesOnDisk  

vi src/BUILD.bazel
```
...
ng_module(
    name = "src",
    srcs = glob(
    ...
    ),
    tsconfig = ":tsconfig.json",
    assets = glob([
    ...
    ]) + ([":styles"] if len(glob(["**/*.scss"])) else []),
    ...
)
...
```

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


#### WORKSPACE : 환경 설정
```
# 이름과 npm directories
workspace (
    name = "bazel-sample"
    managed_directories = { "@npm": "node_modules"]}
)

# ESModule imports (and TypeScript imports) can be absolute starting with the workspace name.
# nodejs의 bazel rule을 다운로드한다. typescript, karma, protractor 등의 기본 규칙이 들어 있다.
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

RULES_NODEJS_VERSION = "1.6.0"
RULES_NODEJS_SHA256 = "f9e7b9f42ae202cc2d2ce6d698ccb49a9f7f7ea572a78fd451696d03ef2ee116"
http_archive(
    name = "build_bazel_rules_nodejs",
    sha256 = RULES_NODEJS_SHA256,
    url = "https://github.com/bazelbuild/rules_nodejs/releases/download/%s/rules_nodejs-%s.tar.gz" % (RULES_NODEJS_VERSION, RULES_NODEJS_VERSION),
)

# Rules for compiling sass
RULES_SASS_VERSION = "1.24.0"
RULES_SASS_SHA256 = "77e241148f26d5dbb98f96fe0029d8f221c6cb75edbb83e781e08ac7f5322c5f"
http_archive(
    name = "io_bazel_rules_sass",
    sha256 = RULES_SASS_SHA256,
    strip_prefix = "rules_sass-%s" % RULES_SASS_VERSION,
    urls = [
        "https://github.com/bazelbuild/rules_sass/archive/%s.zip" % RULES_SASS_VERSION,
        "https://mirror.bazel.build/github.com/bazelbuild/rules_sass/archive/%s.zip" % RULES_SASS_VERSION,
    ],
)

load ( "@ build_bazel_rules_nodejs // : index.bzl", "node_repositories", "yarn_install")

node_repositories (...)
yarn_install (...)
여기서 node를 Isolate 환경에서 설치하여 yarn 설치를합니다. 이 단계 node의 버전 등 package.json 파일을 지정할 수도 있습니다.



```

## appendix

#### reference.site

* Bazel  
https://www.bazel.build/  

* Installing Bazel on Ubuntu  
https://docs.bazel.build/versions/master/install-ubuntu.html  

* bazelbuild / rules_nodejs  
https://github.com/bazelbuild/rules_nodejs  

* angular / angular-bazel-example  
https://github.com/angular/angular-bazel-example  

+ Angular + Bazel : 준비 중!  
https://dev.to/thisdotmedia/angular-bazel-getting-ready-4b0i
