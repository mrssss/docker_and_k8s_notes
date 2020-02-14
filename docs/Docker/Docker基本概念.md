# Docker基本概念
---

## Docker的核心技术
Docker核心是一个操作系统级虚拟化方法。可以分为以下四个方面：

* 隔离性
* 可配额/可度量
* 便携性
* 安全性

### 隔离性：Linux Namespace（ns）
每个用户实例之间相互隔离，互不影响。

* pid namespace：不同用户的进程就是通过pid namespace隔离开的，且不同namespace中可以有相同的pid。
* net namespace：网络隔离，每个net namespace有独立的network devices， IP addresses， IP routing tables，`/proc/net`目录。这样每个container的网络就能隔离开。docker默认采用veth的方式将container中的虚拟网卡同host上的一个docker bridge：docker0连接在一起。
* ipc namespace：container中进程交互还是采用进程间通信的方法，包括常见的信号量、消息队列和共享内存。
* mnt namespace：类似chroot，将一个进程放到一个特定的目录执行。mnt namespace允许不同namespace的进程看到的文件结构不同。
* uts namespace：UTS（UNIX Time-sharing system） namespace允许每个container拥有独立的hostname和domain name，使其在网络上可以被视作一个独立的结点而非Host上的一个进程。
* user namespace：每个container可以有不同的user和group id，也就是说可以在container内部用container内部的用户执行程序，而非Host上的用户。

### 可配额/可度量：Control Groups（cgroups）
cgroups实现了对资源的配额和度量。cgroups可以限制的资源：
* blkio：块I/O设备
* cpu
* cpuacct
* cpuset
* devices
* freezer
* memory
* net_cls
* ns名称空间子系统

### 便携性：AUFS
联合文件系统

### 安全性：AppArmor， SELinux，GRSEC
1. 有kernel namespace和cgroups实现的Linux系统固有的安全标准
2. Docker Deamon的安全接口
3. Linux本身的安全加固解决方案

## Docker概念

### 容器
容器通过对操作系统的资源访问进行限制，构建成独立的资源池，让应用运行在一个相对隔离的空间里，同时容器也可以进行通信。  

比虚拟机技术更轻量级，资源消耗少。于主机共享操作系统内核。

容器是被隔离的进程，它的本质依然是进程。

容器还可以理解成有Docker镜像创建的运行实例。

### 镜像
一个只读的模板，一个独立的文件系统，包括运行容器所需的数据，可以用来创建新的容器。  

基于Dockerfile构建。

### 仓库
存放Docker镜像的服务器，相当于github之于git是代码仓库，Docker的仓库是镜像仓库。

## docker配置
### 问题1: 每次需要sudo才能执行docker命令
这是因为当前用户可能没有权限访问Docker守护进程绑定的Unix套接字，该套接字属于root用户。  
我们可以将需要执行docker命令的用户添加到用户组docker中。该用户组在安装docker后会自动创建。

```sh
$ sudo gpasswd -a username docker
# 也可以使用下面这条命令
$ sudo usermod -aG docker username
# 在终端重启后生效
```

### 问题2:配置国内镜像仓库
编辑`/etc/docker/daemon.json`
添加下面的内容：

```json
{
  "registry-mirrors": ["https://n6syp70m.mirror.aliyuncs.com"]
}
```

然后重启docker服务：

```bash
$ sudo service docker restart
```

## 运行docker容器

docker版Hello world
```bash
$ docker container run hello-world
```
