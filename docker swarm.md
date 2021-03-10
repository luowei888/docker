### Docker swarm

##### 工作模式

![image-20210123204850553](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210123204850553.png)

##### #查看帮助

```shell
[root@localhost ~]# docker swarm --help

Usage:  docker swarm COMMAND

Manage Swarm

Commands:
  ca          Display and rotate the root CA
  init        Initialize a swarm
  join        Join a swarm as a node and/or manager
  join-token  Manage join tokens
  leave       Leave the swarm
  unlock      Unlock swarm
  unlock-key  Manage the unlock key
  update      Update the swarm

Run 'docker swarm COMMAND --help' for more information on a command.


[root@localhost ~]# docker swarm init --help

Usage:  docker swarm init [OPTIONS]

Initialize a swarm

Options:
      --advertise-addr string                  Advertised address (format:
                                               <ip|interface>[:port])
      --autolock                               Enable manager autolocking
                                               (requiring an unlock key to
                                               start a stopped manager)
      --availability string                    Availability of the node
                                               ("active"|"pause"|"drain")
                                               (default "active")
      --cert-expiry duration                   Validity period for node
                                               certificates (ns|us|ms|s|m|h)
                                               (default 2160h0m0s)
      --data-path-addr string                  Address or interface to use for
                                               data path traffic (format:
                                               <ip|interface>)
      --data-path-port uint32                  Port number to use for data path
                                               traffic (1024 - 49151). If no
                                               value is set or is set to 0, the
                                               default port (4789) is used.
      --default-addr-pool ipNetSlice           default address pool in CIDR
                                               format (default [])
      --default-addr-pool-mask-length uint32   default address pool subnet mask
                                               length (default 24)
      --dispatcher-heartbeat duration          Dispatcher heartbeat period
                                               (ns|us|ms|s|m|h) (default 5s)
      --external-ca external-ca                Specifications of one or more
                                               certificate signing endpoints
      --force-new-cluster                      Force create a new cluster from
                                               current state
      --listen-addr node-addr                  Listen address (format:
                                               <ip|interface>[:port]) (default
                                               0.0.0.0:2377)
      --max-snapshots uint                     Number of additional Raft
                                               snapshots to retain
      --snapshot-interval uint                 Number of log entries between
                                               Raft snapshots (default 10000)
      --task-history-limit int                 Task history retention limi 
```

```shell
[root@localhost ~]# docker swarm init --advertise-addr 192.168.31.129   #加入节点
```

![image-20210123202338419](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210123202338419.png)

 



docker swarm init 初始化节点

```shell
1.获取令牌
2.docker swarm join-token manager
3.docker swarm join-token worker
```

加入一个工作节点。

![image-20210123202729483](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210123202729483.png)

![image-20210123202818370](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210123202818370.png)

```shell
[root@localhost ~]# docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1w5y9o1qh1vqz0v8kkk1izbhf8qq3pgzze6i4mw8mwsc5e5q0z-1ydckluezbmjtcowxley2qba5 192.168.31.129:2377

[root@localhost ~]# docker swarm join --token SWMTKN-1-1w5y9o1qh1vqz0v8kkk1izbhf8qq3pgzze6i4mw8mwsc5e5q0z-1ydckluezbmjtcowxley2qba5 192.168.31.129:2377
This node joined a swarm as a worker.
#在本机上执行 离开swarm 
#docker swarm leave --force  #node   #在要被删除的主机上执行
#docker node rm -f  <node>    #manager  在manager节点上执行
[root@localhost ~]# docker swarm leave -f   #在要被删除的主机上执行
Node left the swarm.

[root@localhost ~]# docker swarm join-token manager
[root@localhost ~]# docker swarm join --token SWMTKN-1-4tpix40hzaek4ehp12cotaxbvhbvn3mlho9wtm5jrqcxqu31gn-chlsp06abtu3z48q87f2rd8hj 192.168.31.129:2377
This node joined a swarm as a manager.
```

### Raft协议

1.双主双从：假设一个节点挂了，但是其他节点是否可以用。

2.Raft协议：保证大多数节点存活才可以用，集群至少大于3台

实验：

1.将docker 其中manager节点停止，宕机，另外一个主节点也不能用了。

2.三主一从   



总结：集群 ，可用，3个主节点，  >1台管理节点存货

raft协议：保证大多数节点存活，才可以使用

体会：

以后告别 docker run

docker compose up 启动一个项目，单机

集群：swarm       docker service

容器--》服务

体验：创建服务，动态扩展服务，动态更新服务

![image-20210123212920817](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210123212920817.png)

灰度发布：金丝雀发布

docker run 容器启动，不支持扩缩容j

docker service  启动服务，支持扩缩容 滚动更新

```shell
#单台服务
[root@localhost ~]# docker service create --mode global  -p 8888:80 --name my-nginx nginx
[root@localhost ~]# docker service inspect my-nginx
[root@localhost ~]# docker service update --replicas 3 my-nginx  #搭建多个服务
等价于
[root@localhost ~]# docker service scale my-nginx=5
```

服务，集群的任意接单都可以访问，服务可以有多个副本动态扩缩容实现高可用



概念总结：

swarm

集群的管理核编排，docker 可以初始化一个swarm集群，其他节点可以加入

node:

就是一个docker节点，多个节点就组成一个网络集群，

service

任务，可以在管理节点或者工作节点运行，

task

容器内的命令，

框架图

![image-20210123215826353](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210123215826353.png)

--mode replicated 只在单个节点上部署

--mode global 全部节点部署

![image-20210123220212146](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210123220212146.png)

### 拓展

在 Swarm Service 中有三个重要的网络概念：

- **Overlay networks** 管理 Swarm 中 Docker 守护进程间的通信。你可以将服务附加到一个或多个已存在的 `overlay` 网络上，使得服务与服务之间能够通信。

- **ingress network** 是一个特殊的 `overlay` 网络，用于服务节点间的**负载均衡**。当任何 Swarm 节点在发布的端口上接收到请求时，它将该请求交给一个名为 `IPVS` 的模块。`IPVS` 跟踪参与该服务的所有IP地址，选择其中的一个，并通过 `ingress` 网络将请求路由到它。
   初始化或加入 Swarm 集群时会自动创建 `ingress` 网络，大多数情况下，用户不需要自定义配置，但是 docker 17.05 和更高版本允许你自定义。

- **docker_gwbridge**是一种桥接网络，将 `overlay` 网络（包括 `ingress` 网络）连接到一个单独的 Docker 守护进程的物理网络。默认情况下，服务正在运行的每个容器都连接到本地 Docker 守护进程主机的 `docker_gwbridge` 网络。
   `docker_gwbridge` 网络在初始化或加入 Swarm 时自动创建。大多数情况下，用户不需要自定义配置，但是 Docker 允许自定

  义。

Docker stack

Docker Secret

Docker Config

学习方式：官方文档和命令帮助文档，--help



### 问

docker swarm 的网络模式？？？