# Docker容器管理
---

## 查看帮助
```sh
$ docker -h
```

## docker命令格式
```sh
# Usage:	docker [OPTIONS] COMMAND
```

其中COMMAND被分成了两种：
* Management commands
* Commands

docker CLI中的顶级命令太多，不方便记忆，所以被分组成了Management Commands。  

可以理解为Management Commands是docker命令执行的对象。

docker的help文档讲的听清楚的。有问题先查文档。

## 查看系统信息
```sh
$ docker system info
# 或者使用
$ docker info
```

## 容器生命周期管理
### 创建容器1
```sh
# Management Commands
$ docker container run [OPTIONS] IMAGE [COMMAND [ARGS...]]

# 旧命令
$ docker run [OPTIONS] IMAGE [COMMAND [ARGS...]]
```

Options:
* `-i, --interactive`交互模式
* `-t, --tty`分配一个终端
* `--rm`在容器退出后自动删除
* `-p`将容器的端口映射到主机
* `-d`后台运行
* `-v, --volume`指定数据卷

```sh
$ docker container run busybox echo "hello world"
```
### 创建容器2

```sh
# Management Commands
$ docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]
# 旧命令
$ docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
```
Options:
* `--name`容器名
* `--hostname`容器的主机名
* `--mac-address`MAC地址
* `--ulimit`Ulimit选项

该命令会在指定的镜像IMAGE上创建一个可写容器层，并准备运行指定的命令，这个时候容器并没有被运行。

### 启动容器
对已经Create但还没有运行的容器执行start命令，让容器执行。

```sh
# Management Commands
$ docker container start [OPTIONS] CONTAINER [CONTAINER...]
# Commands
$ docker start [OPTIONS] CONTAINER [CONTAINER...]
```

### 停止容器

```sh
# Management Commands
$ docker container stop CONTAINER [CONTAINER...]
# Commands
$ docker stop CONTAINER [CONTAINER...]
```

### 重启容器

```sh
# Management Commands
$ docker container restart CONTAINER [CONTAINER...]
# Commands
$ docker restart CONTAINER [CONTAINER...]
```

### 容器的暂停
```sh
# Management Commands
$ docker container pause CONTAINER [CONTAINER...]
# Commands
$ docker pause CONTAINER [CONTAINER...]
```

### 容器的恢复

```sh
# Management Commands
$ docker container unpause CONTAINER [CONTAINER...]
# Commands
$ docker unpause CONTAINER [CONTAINER...]
```

### 查看容器列表
```sh
# Management Commands
$ docker container ls [OPTIONS]
# Commands
$ docker ps [OPTIONS]
```

OPTIONS:
* `-a`显示所有的容器
* `-q`仅显示ID
* `-s`显示总的文件大小

### 连接到正在运行中的容器
将本地标准输入输出流连接到一个运行中的容器
```sh
# Management Commands
$ docker container attach [OPTIONS] CONTAINER
# Commands
$ docker attach [OPTIONS] CONTAINER
```

### 查看容器的元数据
```sh
# Management Commands
$ docker container inspect [OPTIONS] CONTAINER [CONTAINER...]
# Commands
$ docker inspect [OPTIONS] CONTAINER [CONTAINER...]
```

### 容器的日志管理
获取容器的输出信息
```sh
# Management Commands
$ docker container logs [OPTIONS] CONTAINER
# Commands
$ docker logs [OPTIONS] CONTAINER 
```

OPTIONS:
* `-t, --timestamps`显示时间戳
* `-f`实时输出

### 显示容器中的进程信息
```sh
# Management Commands
$ docker container top CONTAINER
# Commands
$ docker top CONTAINER
```

### 查看文件修改
查看容器相对于镜像的文件系统来说做了哪些修改
```sh
# Management Commands
$ docker container diff CONTAINER 
# Commands
$ docker diff CONTAINER 
```

### 容器中执行命令
在运行中的容器中执行命令
```sh
$ docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

### 删除容器
```sh
# Management Commands
$ docker container rm [OPTIONS] CONTAINER [CONTAINER...]
# Commands
$ docker rm [OPTIONS] CONTAINER [CONTAINER...]
```
OPTIONS:
* `-f`强制删除运行中的容器