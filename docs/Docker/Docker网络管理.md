# Docker网络管理
---

Docker的网络子系统是可插拔的，使用drivers。有五种默认的网络驱动：
* bridge
    * 默认的网络驱动。如果你没有指定网络驱动，会使用这个驱动。
    * 适用于多个有互相通信需求的container运行在一个主机上
* host
    * 使用主机网络，不使用docker的网络隔离功能，直接使用主机的网络
    * 适用于网络功能不需要被隔离，而其他的部分需要被隔离的容器
* overlay
    * overlay网络会将多个docker的Daemon连接在一起，并且允许swarm的services之间互相通信。
    * 适用于多个运行在不同的docker主机上的有通信需求的容器
* macvlan
    * 允许你给容器指定一个MAC地址，使其看上去有一个物理网卡。
    * 适用于容器需要看上去像是一个物理主机的时候。
* none
    * 禁止所有的网络

用户还可以安装docker的第三方网络插件，来支持更多的网络驱动

## 查看网络
```sh
$ docker network ls
```


### bridge
桥接网络，在安装docker之后会创建一个桥接网络，名为docker0。只有在同一台主机上的container才能够通过bridge网络通信。

#### 使用默认的桥接网络
```sh
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "6160a3c4b610907493f721e4e0221b4e6a4e894af9142bcf1d0143f4422ed23a",
        "Created": "2020-02-16T15:16:45.107374995+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
$ ifconfig
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:e9:d2:3a:2c  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

默认情况下，创建一个新的容器都会自动连接到bridge网络。

```sh
$ docker run -it ubuntu /bin/bash
$ docker network inspect bridge
...
"Containers": {
            "806a73922c60c301ef191e4727fab36923a868bfbcfde838885a7e9c50ea4238": {
                "Name": "dreamy_austin",
                "EndpointID": "a9c13815e2bf488281949faf0038c4cb7df277d066755448101ed494af26a2ea",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
...
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
806a73922c60        ubuntu              "/bin/bash"         2 minutes ago       Up 2 minutes                            dreamy_austin
```

安装ifconfig
```sh
$ apt-get update
$ apt-get install net-tools
```

在容器中查看网卡信息
```sh
$ docker attach 806
root@806...: /# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 5034  bytes 18027836 (18.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3402  bytes 188255 (188.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

#### 使用用户定义的桥接网络

```sh
# 创建桥接网络
$ docker network create --driver bridge alpine-net
# 显示网络
$ docker network ls
docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
0f5e03037991        alpine-net          bridge              local
6160a3c4b610        bridge              bridge              local
f42beaf10829        docker_gwbridge     bridge              local
03ab9c7f2d13        host                host                local
59a32f33acda        none                null                local
$ docker network inspect alpine-net
[
    {
        "Name": "alpine-net",
        "Id": "0f5e03037991b819d904c558163e3b48c0854709073cc6993f1af1bd4cff2829",
        "Created": "2020-02-16T21:48:55.175195644+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

启动四个container让它们的网络选项都选alpine-net
```sh
$ docker run -dit --name alpine1 --network alpine-net alpine ash
$ docker run -dit --name alpine2 --network alpine-net alpine ash
$ docker run -dit --name alpine3 alpine ash
$ docker run -dit --name alpine4 --network alpine-net alpine ash
# 把container alpine4加入bridge网络
$ docker network connect bridge alpine4
```

```sh
docker network inspect alpine-net
[
    {
        "Name": "alpine-net",
        "Id": "0f5e03037991b819d904c558163e3b48c0854709073cc6993f1af1bd4cff2829",
        "Created": "2020-02-16T21:48:55.175195644+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2721bea542e7bfe26ea3ec7534b7426f98266fff15ed03e929cc8a20ee06ced2": {
                "Name": "alpine2",
                "EndpointID": "703af3a34678a08a70f9c8eaaacd4d14fb82866f7bac13761cfe5882f43efe74",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "ab5affcfe88f94f759f463f3f6d39cd577726e9a406dcf46bd8ee85c2e8163c8": {
                "Name": "alpine1",
                "EndpointID": "b649a8bb5ab045d683370b6f06025d41dfcb3873b6698d845ef266a2ba5ef8b2",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "c60451b72c86721bb9dffd8e431a5b6b7d4d1837e9afa3aa80826692e87be784": {
                "Name": "alpine4",
                "EndpointID": "dcfcbfe2cc049c0e81e697f73c3068842f663510760befd707e2a669321d18da",
                "MacAddress": "02:42:ac:13:00:04",
                "IPv4Address": "172.19.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
¥ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "6160a3c4b610907493f721e4e0221b4e6a4e894af9142bcf1d0143f4422ed23a",
        "Created": "2020-02-16T15:16:45.107374995+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4c15e07572a88680cb657016cf228a955fcc1861c891bc9baa87e16c202678a5": {
                "Name": "alpine3",
                "EndpointID": "1d8449f253cc8fc6d54e9d984b7091613323779be7695f96f7b60d1007d61fd5",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "806a73922c60c301ef191e4727fab36923a868bfbcfde838885a7e9c50ea4238": {
                "Name": "dreamy_austin",
                "EndpointID": "a9c13815e2bf488281949faf0038c4cb7df277d066755448101ed494af26a2ea",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "c60451b72c86721bb9dffd8e431a5b6b7d4d1837e9afa3aa80826692e87be784": {
                "Name": "alpine4",
                "EndpointID": "2c70803552eedcb42709b0c7e95fdffdde687a1d5cc52e5234281996b21cccef",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "6160a3c4b610907493f721e4e0221b4e6a4e894af9142bcf1d0143f4422ed23a",
        "Created": "2020-02-16T15:16:45.107374995+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "4c15e07572a88680cb657016cf228a955fcc1861c891bc9baa87e16c202678a5": {
                "Name": "alpine3",
                "EndpointID": "1d8449f253cc8fc6d54e9d984b7091613323779be7695f96f7b60d1007d61fd5",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "806a73922c60c301ef191e4727fab36923a868bfbcfde838885a7e9c50ea4238": {
                "Name": "dreamy_austin",
                "EndpointID": "a9c13815e2bf488281949faf0038c4cb7df277d066755448101ed494af26a2ea",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "c60451b72c86721bb9dffd8e431a5b6b7d4d1837e9afa3aa80826692e87be784": {
                "Name": "alpine4",
                "EndpointID": "2c70803552eedcb42709b0c7e95fdffdde687a1d5cc52e5234281996b21cccef",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

```sh
docker attach alpine1
/ # ping alpine2
PING alpine2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.249 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.478 ms
^C
--- alpine2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.249/0.363/0.478 ms
/ # ping alpine3
ping: bad address 'alpine3'
/ # ping alpine4
PING alpine4 (172.19.0.4): 56 data bytes
64 bytes from 172.19.0.4: seq=0 ttl=64 time=0.104 ms
64 bytes from 172.19.0.4: seq=1 ttl=64 time=0.084 ms
^C
--- alpine4 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.084/0.094/0.104 ms
```

删除bridge网络
```sh
docker network rm alpine-net
```

### host
容器直接绑定主机网络

```sh
$ docker run --rm -d --network host --name my_nginx nginx
f9b9f70b1bbad20e3fddc5615b1c31e79580969e15f71b36b053d03b1ced57d3
$ sudo netstat -tulpn | grep :80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5615/nginx: master
$ docker stop my_nginx
```

### Overlay
在同一个Overlay network下的container可以互相通信，不论它们是否在同一台主机中。

```sh
# TODO: 写完swarm的笔记之后再加上这一段
[overlay network tutorial](https://docs.docker.com/network/network-tutorial-overlay/)
```

### Macvlan
创建虚拟网卡，让container像是使用物理网卡一样

```sh
$ docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 \
  my-macvlan-net
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
6160a3c4b610        bridge              bridge              local
f42beaf10829        docker_gwbridge     bridge              local
03ab9c7f2d13        host                host                local
37d105df8de2        my-macvlan-net      macvlan             local
59a32f33acda        none                null                local
$ docker run --rm -dit \
  --network my-macvlan-net \
  --name my-macvlan-alpine \
  alpine:latest \
  ash
$ docker container inspect my-macvlan-net
[
    {
        "Id": "9a57453a23f093a1b543e36fe0d3791bd40b37d38ac3339bf5075a3a0c6ac4fa",
        "Created": "2020-02-16T14:34:17.375398193Z",
        "Path": "ash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 5816,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-02-16T14:34:18.015482455Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:196d12cf6ab19273823e700516e98eb1910b03b17840f9d5509f03858484d321",
        "ResolvConfPath": "/var/lib/docker/containers/9a57453a23f093a1b543e36fe0d3791bd40b37d38ac3339bf5075a3a0c6ac4fa/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/9a57453a23f093a1b543e36fe0d3791bd40b37d38ac3339bf5075a3a0c6ac4fa/hostname",
        "HostsPath": "/var/lib/docker/containers/9a57453a23f093a1b543e36fe0d3791bd40b37d38ac3339bf5075a3a0c6ac4fa/hosts",
        "LogPath": "/var/lib/docker/containers/9a57453a23f093a1b543e36fe0d3791bd40b37d38ac3339bf5075a3a0c6ac4fa/9a57453a23f093a1b543e36fe0d3791bd40b37d38ac3339bf5075a3a0c6ac4fa-json.log",
        "Name": "/my-macvlan-alpine",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "my-macvlan-net",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": true,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "shareable",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/asound",
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/f8b2319f6190039e80925e200ff2524b5d55aaed63e4ff355fbc02589518aa7f-init/diff:/var/lib/docker/overlay2/98188954f522d047e589b6dfe6f061ac8d768931000146717fc4bcd3785d600d/diff",
                "MergedDir": "/var/lib/docker/overlay2/f8b2319f6190039e80925e200ff2524b5d55aaed63e4ff355fbc02589518aa7f/merged",
                "UpperDir": "/var/lib/docker/overlay2/f8b2319f6190039e80925e200ff2524b5d55aaed63e4ff355fbc02589518aa7f/diff",
                "WorkDir": "/var/lib/docker/overlay2/f8b2319f6190039e80925e200ff2524b5d55aaed63e4ff355fbc02589518aa7f/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "9a57453a23f0",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "ash"
            ],
            "Image": "alpine:latest",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "8e65f5fdb7a2e710c679a33dc057896f3833856f1bf19fcad93af6a406734764",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/8e65f5fdb7a2",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "my-macvlan-net": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "9a57453a23f0"
                    ],
                    "NetworkID": "37d105df8de2195b7a7e46c0f712fef8100585080cf5519c1b34b04f0ad0cc6d",
                    "EndpointID": "e771a6d6481688f3d7cd26dbd4d8320593b5a219524dca6cd684efed03f9b7e3",
                    "Gateway": "172.16.86.1",
                    "IPAddress": "172.16.86.2",
                    "IPPrefixLen": 24,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:10:56:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
$ docker exec my-macvlan-alpine ip addr show eth0
20: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:10:56:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.86.2/24 brd 172.16.86.255 scope global eth0
       valid_lft forever preferred_lft forever
$ docker exec my-macvlan-alpine ip route
default via 172.16.86.1 dev eth0
172.16.86.0/24 dev eth0 scope link  src 172.16.86.2
$ docker container stop my-macvlan-alpine
$ docker network rm my-macvlan-net
```

