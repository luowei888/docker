## Docker网络

### 理解docker

```shell
#问题：docker是怎么处理容器之间网络访问的
```

```shell
[root@localhost ~]# docker run -d -P --name tomcat01 tomcat

#查看容器内部网络地址，ip addr  发现容器启动的时候会拿到一个eth0 ip地址，docker分配的


[root@localhost ~]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
142: eth0@if143: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@localhost ~]#


#思考 linux可以直接ping通容器内部么？[root@localhost ~]# ping  172.16.0.2
PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
^C
--- 172.16.0.2 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3000ms

[root@localhost ~]# ping  172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.148 ms

#测试得出是可以ping 通容器内部的
```

> 原理

1.我们每启动一个docker容器，docker就会给docker容器分配一个IP，我们只要安装docker，就会就会有一个docke0桥接模式，实用技术时evth-pair技术！！

```shell
#我们发现这个启动容器会生成网卡时一对一对的

#evht-pair 就是一对的虚拟设备接口，他们都是成对出现的，一段连接协议，一段彼此相连
#正因为有这个特性，evth-pair 充当一个桥梁，连接各种虚拟网络设备
#openstck DOCKER,容器之间的连接，ovs的连接，都是使用evth-pair 技术
```

3.我们来测试容器之间互通，

```shell
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS                     NAMES
4026871ca900   tomcat    "catalina.sh run"   10 minutes ago   Up 10 minutes   0.0.0.0:49156->8080/tcp   tomcat02(172.17.0.3)
95487eca764b   tomcat    "catalina.sh run"   22 minutes ago   Up 22 minutes   0.0.0.0:49155->8080/tcp   tomcat01(172.17.0.2)


[root@localhost ~]# docker exec -it tomcat01 ping 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.165 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.073 ms
^C
--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1ms
rtt min/avg/max/mdev = 0.073/0.119/0.165/0.046 ms
#结论：容器之间可以互相ping通
```

网络模型图

![image-20210122011338935](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122011338935.png)

```shell
#结论：tomcat01和tomcat02都公用的 一个路由器docker0
所有的容器不指定网络的情况下，都是docker0路由的 docker会给我们容器分配一个默认可用的IP
```

> #### 小结

Docker使用的是Linux桥接，宿主机中是一个docker容器的网桥docker0.

![image-20210122012549515](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122012549515.png)

Docker中所有的网络接口都是虚拟的，虚拟转发效率高

只要删除容器，对应网卡一对就没了

![image-20210122020502708](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122020502708.png)

### --link

> 思考一个场景，我们编写了一个微服务，容器死掉，IP会变化这个问题，怎么可以通过名字来访问容器？？？

```shell
#实现ping容器名互通

[root@localhost ~]# docker run -itd --name tomcat01 tomcat /bin/bash
a2047a12bbf358540495c9a9344041bb33be5a135b1871b6d924a26a40fc2d5e
[root@localhost ~]# docker run -itd --name tomcat02 tomcat /bin/bash
dc354a3de2544ceab42c26752c68e393f38bc946478023d74b84df26fecbc675
[root@localhost ~]# docker run -itd --name tomcat03 tomcat /bin/bash
d9340e02c4fbd15f17e433448276a32571743c4aca62391f9e92c76f67d69c18
[root@localhost ~]# docker run -d -P --name tomcat04 --link  tomcat02 tomcat
be229d67d0ded7a7044ca5b5d8d9f9a5d87bfe037ebe25f9b6ab72ee7c28c5db
[root@localhost ~]# docker exec -it tomcat02 ping tomcat04
ping: tomcat04: Name or service not known
[root@localhost ~]# docker exec -it tomcat04 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.092 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.049 ms
^C
--- tomcat02 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 0.049/0.066/0.092/0.020 ms
[root@localhost ~]#


#反向不能通 tomcat02 ping tomcat04
```

#### 探究：inspect

```shell
#查看tomcat04 /etc/hosts文件，里面配置了tomcat02和他的ip地址绑定。

[root@localhost ~]# docker exec -it tomcat04 cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      tomcat02 dc354a3de254
172.17.0.5      be229d67d0de
[root@localhost ~]#


```

--link  就是在我们的hosts配置中增加了 172.17.0.3 tomacat02 dc354a3de254

#### 自定义网络

------

```shell
#查看所有网络
[root@localhost ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
f96e13b775b6   bridge    bridge    local
11eb396b3667   host      host      local
5f06da7515b3   none      null      local
```

##### 网络模式

bridge :桥接 dockers(默认)

none :不配置网络

host:和宿主机共享网络

container :容器网络联通（用的少）



```shell
#我们直接启动的命令 --net bridge ,而这个就是我们的docker0
[root@localhost ~]# docker run -d -P --name tomcat01  tomcat
[root@localhost ~]# docker run -d -P --name tomcat01 --net bridge tomcat
#
#docker0的特点，默认，域名不能访问， --link 可以通过连接


```

```shell
#我们可以自定义一个网络。
[root@localhost ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway=192.168.0.1 mynet
[root@localhost ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "5cc285ff4fe98326fe98552b7b8e19a62c209b895d7e9fb472491eeba9153cc9",
        "Created": "2021-01-22T03:17:14.517296862+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
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



```shell
#将容器网络指定为自定义网络mynet
[root@localhost ~]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
815557c16e8672b60a03eae3e90a52a972fac753863adbe450769a87fa3de8ea
[root@localhost ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "5cc285ff4fe98326fe98552b7b8e19a62c209b895d7e9fb472491eeba9153cc9",
        "Created": "2021-01-22T03:17:14.517296862+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
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
            "815557c16e8672b60a03eae3e90a52a972fac753863adbe450769a87fa3de8ea": {
                "Name": "tomcat-net-01",
                "EndpointID": "d66bbb67d5228cba785717b17d7a9fddb19db658eb900e1c97ef23268493516a",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
[root@localhost ~]# 
```

```shell
[root@localhost ~]# docker run -d -P --name tomcat-net-02 --net mynet tomcat
7d548fba886f168f9f3cfa873d9c27337c2841081701212ee48f9370ac1eed59
[root@localhost ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "5cc285ff4fe98326fe98552b7b8e19a62c209b895d7e9fb472491eeba9153cc9",
        "Created": "2021-01-22T03:17:14.517296862+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
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
            "7d548fba886f168f9f3cfa873d9c27337c2841081701212ee48f9370ac1eed59": {
                "Name": "tomcat-net-02",
                "EndpointID": "b6c2f8a018e192946b53a5f258a17a96142008b3115b36ab42f14399688a59e9",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "815557c16e8672b60a03eae3e90a52a972fac753863adbe450769a87fa3de8ea": {
                "Name": "tomcat-net-01",
                "EndpointID": "d66bbb67d5228cba785717b17d7a9fddb19db658eb900e1c97ef23268493516a",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

```shell
#测试结果
[root@localhost ~]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.116 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.098 ms
```

```shell
#结论：自定义网络，不需要--link 可以ping 容器名字
#我们自定义的网络docker已经帮我们维护好了对应得关系，推荐建议使用自定义网络。
```

##### 自定义网络的优势

redis- 不同集群使用不同的网络，保证集群健康

mysql- 不同集群使用不同的网络，保证集群健康

#### 网络联通

![image-20210122033831308](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122033831308.png)

```shell

#测试环境
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS                     NAMES
2d9abbc86523   tomcat    "catalina.sh run"   8 seconds ago    Up 7 seconds    0.0.0.0:49164->8080/tcp   tomcat04
273aeb68b3db   tomcat    "catalina.sh run"   18 seconds ago   Up 17 seconds   0.0.0.0:49163->8080/tcp   tomcat03
7d548fba886f   tomcat    "catalina.sh run"   21 minutes ago   Up 21 minutes   0.0.0.0:49159->8080/tcp   tomcat-net-02
815557c16e86   tomcat    "catalina.sh run"   23 minutes ago   Up 23 minutes   0.0.0.0:49158->8080/tcp   tomcat-net-01

```

#测试打通tomcat01 tomcat02 到tomcat03 tomcat04 

```shell
[root@localhost ~]# docker network connnect --help
Commands:
  connect     Connect a container to a network
[root@localhost ~]# docker network connect mynet tomcat01
#发现docker 把tomcat01加入了mynet这个网络 在tomcat01里加了一块mynet网段的网卡。

[root@localhost ~]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
175: eth0@if176: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
179: eth1@if180: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:c0:a8:00:04 brd ff:ff:ff:ff:ff:ff link-netnsid 0
   # inet 192.168.0.4/16 brd 192.168.255.255 scope global eth1
       valid_lft forever preferred_lft forever

```

·