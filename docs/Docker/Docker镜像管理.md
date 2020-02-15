# Docker镜像管理
---

## 查看镜像列表
```sh
# Management Commands
$ docker image ls
# Commands
$ docker images
```

## 查看镜像详细信息
```sh
# Management Commands
$ docker image inspect IMAGE
# Commands
$ docker inspect IMAGE
```

## 搜索镜像
```sh
$ docker search IMAGE
```

## 拉取镜像
```sh
# Management Commands
$ docker image pull [OPTIONS] name[:TAG|@DIGSET]
# Commands
$ docker pull [OPTIONS] NAME[:TAG|@DIGSET]
```

OPTIONS:
* `-a` 下载仓库中的所有镜像，即下载整个存储库

对于pull下来的镜像来说，其默认的保存路径为`/var/lib/docker`

## 构建镜像
### commit
创建一个容器，在容器中进行修改，然后将修改提交到一个新的镜像中
```sh
# Management Commands
$ docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
# Commands
$ docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

### build
从一个Dockerfile文件中自动读取指令构建一个新的镜像。
```sh
# Management Commands
$ docker image build [OPTIONS] PATH | URL
```
OPTIONS：
* `-t`指定新的镜像IMAGENAME[:TAG]

Dockerfile基本语法：
```Dockerfile
# 注释
INSTRUCTION arguments
```

Dockerfile指令一般包含下面几个部分：
1. 基础镜像：FROM指令。一个Dockerfile必须以FROM指令启动
2. 维护者信息：MAINTAINER指令。
3. 镜像操作指令：对基础镜像进行改造的命令，常见的RUN指令
4. 容器启动命令：基于该镜像的容器启动时需要执行哪些命令，常见的是CMD命令或ENTRYPOINT

```Dockerfile
FROM ubuntu:14.04

MAINTAINER qniu/niuqijun0702@qq.com

RUN apt-get -y update && \
    apt-get install -y apache2

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

构建镜像
```sh
# 默认会在PATH寻找Dockerfile，也可以使用-f指定文件名，和PATH组合起来：PATH/FILENAME
$ docker image build -t qniu/apache:1.0 .
```

## 删除镜像
如果有容器使用该镜像，只有删除容器了之后才能够删除镜像
```sh
# Management Commands
$ docker image rm ubuntu:latest
# Commands
$ docker rmi ubuntu:latest
```
