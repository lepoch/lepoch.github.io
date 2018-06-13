---
title: Go小记-初识
date: 2018-04-04 17:00:00
tags:
 - Go
 - 笔记

categories:
 - Go

---
整理的一些小记

<!-- MORE -->
## [dep](https://github.com/golang/dep)
```
brew install dep
```

## 情景
- gitlab

## Go tools

### Lint
语法检测与建议

#### 安装
```bash
go get -u golang.org/x/lint/golint
```

#### 无梯子的安装
/tools/init.sh
```bash
#!/usr/bin/env bash


## 安装lint，其他需要翻墙的，大多也能这么安装
mkdir -p $GOPATH/src/golang.org/x
if [ ! -d $GOPATH/src/golang.org/x/tools ]; then
	cd $GOPATH/src/golang.org/x
	git clone https://github.com/golang/tools.git
	git clone https://github.com/golang/lint.git

	go get golang.org/x/lint/golint
fi

```

#### 使用
```bash
golint -set_exit_status $(go list ./... | grep -v /vendor/)
```

### Unit test  
单元测试
#### 使用
每个目录都执行 go test 方法
```bash
go test -short $(go list ./... | grep -v /vendor/)
```

### Code coverage
这个可以计算单元测试覆盖率，添加已下脚本文件

#### 使用
/tools/coverage.sh
```bash
#!/usr/bin/env bash
PKG_LIST=$(go list ./... | grep -v /vendor/) 
for package in ${PKG_LIST}; do
    go test -covermode=count -coverprofile "cover/${package##*/}.cov" "$package" ;
done
tail -q -n +2 cover/*.cov >> cover/coverage.cov
go tool cover -func=cover/coverage.cov
```

#### 查看
```
go tool cover -html=cover/coverage.cov -o coverage.html
```

### 写个makefile，来集中调用这些工具 
#### 项目下创建Makefile文件    
一些语法需要注意下：
- 行首的缩进必须是tab


```makefile
PROJECT_NAME := "maozhuaV4"
PKG := "$(PROJECT_NAME)"
PKG_LIST := $(shell go list ${PKG}/... | grep -v /vendor/)
GO_FILES := $(shell find . -name '*.go' | grep -v /vendor/ | grep -v _test.go)
PATH  := $(PATH):${GOPATH}/bin
SHELL := env PATH=$(PATH) /bin/bash

.PHONY: all dep build clean test coverage coverhtml lint

all: build

lint: ## Lint the files
	@golint -set_exit_status ${PKG_LIST}

test: ## Run unittests
	@go test -short ${PKG_LIST}

race: dep ## Run data race detector
	@go test -race -short ${PKG_LIST}

msan: dep ## Run memory sanitizer
	@go test -msan -short ${PKG_LIST}

coverage: ## Generate global code coverage report
	./tools/coverage.sh;

coverhtml: ## Generate global code coverage report in HTML
	./tools/coverage.sh html;

dep: ## Get the dependencies
    ./tools/init.sh;
    ## @go get -v -d ./...

build: dep ## Build the binary file
	@go build -i -v $(PKG)

clean: ## Remove previous build
	@rm -f $(PROJECT_NAME)

help: ## Display this help screen
	@grep -h -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
    
```

#### 使用
``` bash
make help
make lint
make dep
make coverage
## ...

```

## gitlab的CI/CD  
整理好了这些工具，那怎么自动调用呢？ 如gitlab有pipelines、drone等。  
这里使用的是gitlab。

### 创建 .gitlab-ci.yml 
[官方文档](https://docs.gitlab.com/ee/ci/yaml/README.html)
[变量](https://docs.gitlab.com/ee/ci/variables/README.html)

```YAML
image: golang:1.10
cache:
  paths:
    - /apt-cache
    - /go/src/github.com
    - /go/src/golang.org
    - /go/src/google.golang.org
    - /go/src/gopkg.in
stages:
  - test
  - build
before_script:
  - mkdir -p /go/src/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME /go/src/_/builds/$CI_PROJECT_NAMESPACE
  - cp -r $CI_PROJECT_DIR /go/src/$CI_PROJECT_NAME
  - ln -s /go/src/$CI_PROJECT_NAME /go/src/_/builds/$CI_PROJECT_NAMESPACE/$CI_PROJECT_NAME
  - make dep
unit_tests:
  stage: test
  script:
    - make test
race_detector:
  stage: test
  script:
    - make race
#memory_sanitizer:
#  stage: test
#  script:
#    - make msan
code_coverage:
  stage: test
  script:
    - make coverage
code_coverage_report:
  stage: test
  script:
    - make coverhtml
  only:
  - master
lint_code:
  stage: test
  script:
    - make lint
build:
  stage: build
  script:
    - make
```


### [安装Runner](https://docs.gitlab.com/runner/)  
上述只是任务，具体在哪跑，得另行指定
#### k8s  
#### 手动安装  
登录gitlab服务器，执行
```bash
sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64


```


### 镜像build push  
#### docker in docker  
上面的runner用的executor用的是docker，那么问题来了，要怎么把生成好的镜像推送到镜像仓库呢？
- 在容器中再安装docker
- run的时候传入： `docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) -ti google/golang /bin/bash`

gitlab-runner提供了[方法](https://docs.gitlab.com/runner/executors/docker.html#use-docker-in-docker-with-privileged-mode)

1、找到配置文件
```bash
gitlab-runner list
```

/etc/gitlab-runner/config.toml 中修改
```
[[runners]]
  executor = "docker"
  [runners.docker]
    privileged = true
```

2、建个镜像，把go环境也搭好，这里镜像就叫mygolang了
Dockerfile
```yaml 
FROM docker
MAINTAINER lepoch <ll-xzx@qq.com>

#使用 ali 仓库地址
RUN echo -e "http://mirrors.aliyun.com/alpine/latest-stable/main\n\
http://mirrors.aliyun.com/alpine/latest-stable/community" > /etc/apk/repositories


RUN apk update && apk add curl ca-certificates wget bash go1.10 git make && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    rm -rf /root/.cache

RUN set -eux; \
    apk update; \
	apk add --no-cache --virtual .build-deps \
		bash \
		gcc \
		musl-dev \
		openssl \
	; \
	export \
# set GOROOT_BOOTSTRAP such that we can actually build Go
		GOROOT_BOOTSTRAP="$(go env GOROOT)" \
# ... and set "cross-building" related vars to the installed system's values so that we create a build targeting the proper arch
# (for example, if our build host is GOARCH=amd64, but our build env/image is GOARCH=386, our build needs GOARCH=386)
		GOOS="$(go env GOOS)" \
		GOARCH="$(go env GOARCH)" \
		GOHOSTOS="$(go env GOHOSTOS)" \
		GOHOSTARCH="$(go env GOHOSTARCH)" \
        CGO_ENABLED="0" \
	; \
	export PATH="/usr/local/go/bin:$PATH"; \
	go version

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

ENV CGO_ENABLED "0"

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH
```

3、修改yaml文件
```yaml
image: mygolang
# Use the OverlayFS driver for improved performance.
variables:
  DOCKER_DRIVER: overlay

services:
  - docker:dind
```
4、build 和 push

```yaml
build:
  stage: build
  script:
    - docker info
    - docker login -u gitlab-ci-token -p $CI_BUILD_TOKEN registry.maozhuar.com
    - CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main $CI_PROJECT_NAME/
    - docker build -t registry.maozhuar.com/backenddeveloper/maozhuav4:$CI_COMMIT_REF_NAME .
    - docker push registry.maozhuar.com/backenddeveloper/maozhuav4:$CI_COMMIT_REF_NAME
```

### 部署 
以单台机器来说，用ssh远程执行其脚本即可。  

在要部署的机器上，执行
```bash
useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
groupadd docker
usermod -aG docker gitlab-runner
systemctl restart docker
```

#### ssh使用秘钥 [https://docs.gitlab.com/ee/ci/ssh_keys/README.html]
- 创建秘钥对
- 把私钥放入gitlab CI/CD的 secret key
- 把ssh-keyscan的结果也放入 secret key
- 



### 相关网址
- [Go tools & GitLab — how to do Continuous Integration like a boss](https://medium.com/pantomath/go-tools-gitlab-how-to-do-continuous-integration-like-a-boss-941a3a9ad0b6)
- [Runner](https://docs.gitlab.com/runner/)
