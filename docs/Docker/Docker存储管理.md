# 存储管理
---

## 概述
将数据保存在容器中的缺点：

* 当容器不再运行是，我们无法使用数据，当容器被删除时，数据并不会被保存
* 数据保存在容器的可写层中，我们无法轻松的将数据移动到其他地方

对于有些数据文件，我们不将其保存在镜像或容器的可写层中，而是将数据从Docker主机挂载到容器中。  
挂载数据的三种方式：
* 卷（volumes）：卷存储在Docker管理的主机文件系统中的一部分中（`/var/lib/docker/volumes`），完全有Docker管理。
* 绑定挂载（bind mounts）：可以将主机上的文件或目录挂载到容器中。
* 临时文件系统（tmpfs）：仅存储在主机系统的内存中，而不会写入主机的文件系统

无论是哪种方式，数据在容器内看上去都是一样的，它被当作容器文件系统中的目录或单个文件。

## 卷
卷是这三种方式中唯一完全有Docker管理的。它更容易备份或迁移

### 列出本地可用的卷
```sh
$ docker volume ls
```

### 创建卷
```sh
$ docker volume create VOLUME
```

### 用卷启动一个容器
```sh
docker container run [-v|--mount]
```

* `-v`
    * [HOST-DIR:]CONTAINER_DIR[:OPTIOND]
    * HOST-DIR，代表主机上的目录或数据卷的名字，省略该部分时，会自动创建一个匿名数据卷。如果指定的时主机上的目录，需要使用绝对路径
    * CONTAINER-DIR，代表将要挂在到容器中的目录或文件，即表现为容器中的某个目录或文件
    * OPTIONS，代表配置，例如只读（ro），仅能被该容器使用（Z），可以被多个容器使用（z），若个配置项由都好分隔
    * `-v volume1:/volume1:ro,z`
* `--mount`
    * 由多个键值对组成，键值对之间由逗号分隔
    * type，指定类型，bind、volume、tmpfs
    * source，当类型为volume是，指定卷名称，匿名卷时省略该字段。当类型为bind，指定路径。可以使用src缩写。
    * destination，挂载到容器中的路径，可以使用dst或target缩写。
    * ro，配置属性
    * `--mount type=volume,source=volume1,destination=/volume1,ro=true`

启动容器时挂载卷volume1
```sh
# -v
$ docker container run -it -name qniu1 -v volume1:/volume1 --rm ubuntu /bin/bash
# --mount
$ docker container run -it -name qniu2 --mount type=volume,source=volume1,target=/volume1 --rm ubuntu /bin/bash
```

`--mount`的可读性更好，推荐使用

### bind-mounts
把主机的目录挂载到容器中，容器就可以操作和修改主机上该目录的内容。这即是其优点也是其缺点

```sh
# -v
docker container run -it -v /home/qniu:/home/container/qniu --name qniu1 --rm ubuntu /bin/bash
# --mount
docker container run -it --mount type=bind,src=/home/qniu,target=/home/container/qniu --name qniu2 --rm ubuntu /bin/bash
```

### tmpfs
tmpfs只存在主机的内存中，当容器停止时，相应的数据就会被移除。
```sh
$ docker container run --it --mount type=tmpfs,target=/test --name qniu3 --rm ubuntu /bin/bash
```

## 数据卷容器
如果容器之间需要共享一些持续更新的数据，最简单的方式就是使用用户数据卷容器，其他容器通过挂载这个容器实现数据共享，这个挂载数据卷的容器就叫做数据卷容器。  
数据卷容器就是一种普通容器，它专门提供数据卷供其他容器挂载使用

### 创建数据卷容器
```sh
# 创建数据卷vdata
$ docker volume create vdata
# 创建一个挂载了vdata的容器，这个容器就是数据卷容器
$ docker container run -it -v vdata:/vdata -name vdatavolume ubuntu /bin/bash
```

### 从数据卷容器中挂载数据卷
使用`--volumes-from`
```sh
$ docekr container run -it --volumes-from vdatavolume --name qniu1 ubuntu /bin/bash
```

### 数据备份
数据存在数据卷中，如果我们想要备份它，可以采用备份容器的方式
```sh
$ docker container run --volumes-from vdatavolume -v /home/qniu/backup:/backup ubuntu tar cvf /backup/backup.tar /vdata/
```
上面的这个容器挂载了一个数据卷，挂载了一个主机目录，然后将数据卷中的数据打包存储到主机目录中，这样就实现了数据备份。

### 数据恢复
数据备份的逆过程
```sh
$ docker container run --volumes-from vdatacvolume -v /home/qniu/backup:/backup ubuntu tar xvf /backup/backup.tar -C /
```
解压路径为/vdata的上一级目录。
