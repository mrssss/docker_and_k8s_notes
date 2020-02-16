# 编写Dockerfile
---

Dockerfile是一个文本文件，其中包含了我们为了构建Docker镜像而手动执行的所有命令。Docker可以从Dockerfile中读取指令来自动构建镜像。

## 基本语法

### FROM
基础镜像
```dockerfile
FROM ubuntu:14.04
```

### USER
执行指令的用户，后续指令必须用这个用户执行，该用户必须存在。
```sh
USER qniu
```

### WORKDIR
指定工作目录，可以理解为命令执行的当前目录（container中的）
```sh
WORKDIR /
```

### RUN
用于执行命令，有两种形式：
* RUN <command>
* `RUN ["executable", "param1", "param2", ...]`

```sh
RUN apt-get update
```

### CMD
一个Dockerfile中只能有一个CMD，如果有多个会被最后一个覆盖。相当于是默认的docker run中的CMD命令部分，如果一个image指定了CMD可以用`docker run image`来执行。

```Dockerfile
CMD echo "hello world"
```

如果指定了ENTRYPOINT，CMD的指令会作为ENTRYPOINT的参数

### ENTRYPOINT
image的入口

```dockerfile
ENTRYPOINT ["ls", "-a"]
CMD ["-l"]
```

### COPY, ADD
COPY和ADD都用于将文件，目录复制到镜像中。

```dockerfile
ADD <src>... <dest>
ADD ["<SRC>",... "<dest>"]

COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

`<src>`不能超过上下文的路径，必须跟Dockerfile同级或子目录
`<dest>`不需要预先存在，相对路径的话是相对工作目录的

COPY和ADD的不同之处在于，ADD可以添加远程路径的文件，并且`<src>`为可识别的压缩格式，如gzip或tar，ADD会自动将其解压缩为目录。

### ENV
用于设置环境变量

```dockerfile
ENV <key> <value>
ENV <key>=<value> <key>=<value>
```

### VOLUME
创建指定的挂载目录，在容器运行时，将创建相应的匿名卷

```dockerfile
VOLUME /data1 /data2
```

### EXPOSE
指定在容器运行时监听指定的网络端口，只是允许端口暴露出来，只有设置了这个`-p`或`-P`才有用

```dockerfile
EXPOSE port
```

## 从dockerfile创建镜像

```dockerfile
FROM ubuntu:14.04

RUN apt-get update && \
    apt-get install -y openssh-server && \
    mkdir /var/run/sshd

RUN useradd -g root -G sudo qniu && \
    echo "qniu:123456" | chpasswd qniu

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
```

```sh
$ mkdir dir1 && cd dir1
$ vim Dockerfile
$ docker build -t qniu/sshd:0.1 .
```

```sh
$ docker run -itd -p 10001:22 qniu/sshd:0.1
```

