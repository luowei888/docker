## Docker安装

------

​	![image-20210122003216737](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122003216737.png)

Docker 基本概念

![image-20210122110153239](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122110153239.png)



1.官方文档 #注意按照系统安装**https://docs.docker.com/engine/install/centos/ docker** 

```shell
#先卸载旧版本
[root@192 ~]#sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

[root@192 ~]#yum install -y yum-utils
#配置的阿里云的docker仓库地址
[root@192 ~]#yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
[root@192 ~]#yum makecache fast  #清除yum缓存
[root@192 ~]#yum install docker-ce docker-ce-cli containerd.io  #docker-ce是社区版
[root@192 ~]#[root@192 ~]# systemctl  start docker
[root@192 ~]#[root@192 ~]# docker version
```

#### 阿里云镜像加速

------

##### 登陆阿里云--》点击容器镜像服务--》镜像加速器。

​	

```shell
[root@192 ~]#sudo tee /etc/docker/daemon.json <<-'EOF'
	{
	  "registry-mirrors": ["https://0qv1zcsc.mirror.aliyuncs.com"]
	}
	EOF
[root@192 ~]#sudo systemctl daemon-reload
[root@192 ~]#sudo systemctl restart docker
```

#### docker底层原理：

------

`1.docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过socket从客户端访问
​2.dockerserver 接收到docker-client的指令，就会执行命令。
3.dockers为什么比VM快？   我理解：docker和vm是同一层级，但是VM还需要安装系统才能运行虚机，但是docker是直接运行容器应用
​        1.docker有着比虚拟机更少的抽象层，比如（hupervisor和guest os)
​		2.docker 利用的是宿主机的内核，vm需要的GUest os
总结：新建一个容器时，docker不需要加载操作系统内核。虚拟机要加载内核，是分钟级别，docker利用宿主机的操作系统，无需加载操作系统，秒级。`

#### docker常用命令

------

![1](C:\Users\wEI\Desktop\1.png)

##### 帮助命令

```shell

     docker version          #显示docker版本信息
	 docker info             #显示docker的系统信息，包括镜像和容器数量
	 docker --help           #帮助命令
[root@192 ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    bf756fb1ae65   12 months ago   13.3kB

#REPOSITORY ：镜像的仓库源
#TAG        ：镜像的标签
#IMAGE ID   ：镜像的ID
#CREATED  	：镜像的创建时间
#SIZE	    ：镜像的大小
    #可选项
	-a, --all    #列表所有镜像
	-q, --quiet  #只显示镜像ID
	
6.2 docker search 搜索镜像
[root@192 ~]#docker search mysql
```
#帮助文档地址：https://docs.docker.com/engine/reference/commandline/`
#可选项，通过搜索过滤，
   --filter=STARTS=3000 #搜索出来的镜像就是STARS大于3000的镜像。

```shell
[root@192 ~]:docker search mysql --fileter=STARTS=3000
#下载镜像，docker pull 镜像名[:tag]
[root@192 ~]#docker pull mysql
[root@192 ~]# docker pull mysql
Using default tag: latest   #如果不写tag,默认就是latest
latest: Pulling from library/mysql
a076a628af6f: Pull complete   #分层下载，docker image 的核心，联合文件系统
f6c208f3f991: Pull complete 
88a9455a9165: Pull complete 
406c9b8427c6: Pull complete 
7c88599c0b25: Pull complete 
25b5c6debdaf: Pull complete 
43a5816f1617: Pull complete 
69dd1fbf9190: Pull complete 
5346a60dcee8: Pull complete 
ef28da371fc9: Pull complete 
fd04d935b852: Pull complete 
050c49742ea2: Pull complete 
Digest: sha256:0fd2898dc1c946b34dceaccc3b80d38b1049285c1dab70df7480de62265d6213 #签名
Status: Downloaded newer image for mysql:latest 
docker.io/library/mysql:latest #真实地址

#等价于
docker pull mysql  = docker pull docker.io/library/mysql:latest
#指定版本下载
[root@192~]：docker pull mysql:5.7

#删除镜像 docker rmi 
[root@192~]: docker rmi -f 镜像 id #铲除指定镜像 
[root@192~]: docker rmi -f 镜像 id 镜像 id 镜像 id #铲除多个镜像 
[root@192~]: docker rmi -f $(docker images -qa) #铲除全部镜像 
```



##### 6.5 容器命令

```shell
[root@192]#docker pull centos
#docker run [可选参数] image

#可选参数如下：
--name='Name'  #容器名字，tomcat01 tomcat02 用来区分容器
-d             # 后台方式运行
-it            # 使用交互方式运行，进入容器查看内容
-p             # 指定容器的端口 -P 8080:8080
          -p    ip:主机端口：容器端口
		  -p 主机端口：容器端口 
		  -p 容器端口
		  容器端口
-p 随机指定端口

#测试
[root@192 ~]# docker run -it centos /bin/bash
[root@df40c9cf845d /]# ls 
```

##### 6.6列出所有运行的容器

docker ps  命令  
      #列出当前正在运行的容器
	 -a  #列出当前正在运行的容器+带出历史运行的容器
	 -n=？ #显示最近创建的容器
	 -q # 只显示容器编号

```shell
[root@192 ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@192 ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS                       PORTS     NAMES
df40c9cf845d   centos        "/bin/bash"   7 minutes ago    Exited (127) 3 minutes ago             exciting_rubin
2e64e5e01d45   hello-world   "/hello"      36 minutes ago   Exited (0) 36 minutes ago              upbeat_chaum  

```

##### 6.7退出容器

exit #直接容器停止并退出
Ctrl+P+Q # #容器不停止退出

##### 6.8 删除容器

```shell
[root@192 ~]#docker run 容器id                  #删除指定的容器
[root@192 ~]#docker rm -rf $(docker ps -aq)     #删除所有容器
[root@192 ~]#docker ps -a -q | xargs docker rm  #删除所有容器
```



##### 6.9 启动和停止容器的操作

[root@192 ~]#docker start 容器ID
[root@192 ~]#docker restart 容器ID
[root@192 ~]#docker stop 容器id      #停止容器
[root@192 ~]#docker kill 容器ID      #强制停止容器



##### 6.10 #常用的其他命令

#docker run -d 容器名
docker run -d centos   #


？？？ 运行上面命令后，发现centos停止了

#常见的坑，docker 容器使用后台运行，就必须要有一个前台进程，docker发现没有应用，就会自动停止
#nginx 容器启动后，发现自己没有提供服务，就会立刻停止，就没有程序了

##### 6.11 查看日志

```shell

[root@192]# docker logs -f -t --tail 10 容器ID
[root@192]#docker run -d centos /bin/sh -c 'while true;do echo luowei;sleep 1;done'

#显示日志
-tf #显示日志
--tail number(一个数字) #要显示日志条数
```

##### 6.12 查看容器内进程信息 

#docker top 容器ID

```shell
[root@localhost ~]# docker top 9b
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                12376               12356               0                   20:52               pts/0               00:00:00            /bin/bash
#docker inspect 容器ID
[root@localhost ~]# docker inspect 9b
[
    {
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/9b6c571b548e28fe2af17ff88ab2ecf4bb9d1f3ef92695fcb7dd0d153a66d6f2/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/9b6c571b548e28fe2af17ff88ab2ecf4bb9d1f3ef92695fcb7dd0d153a66d6f2/hostname",
        "HostsPath": "/var/lib/docker/containers/9b6c571b548e28fe2af17ff88ab2ecf4bb9d1f3ef92695fcb7dd0d153a66d6f2/hosts",
        "LogPath": "/var/lib/docker/containers/9b6c571b548e28fe2af17ff88ab2ecf4bb9d1f3ef92695fcb7dd0d153a66d6f2/9b6c571b548e28fe2af17ff88ab2ecf4bb9d1f3ef92695fcb7dd0d153a66d6f2-json.log",
    
          }  
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/7d6ff5f6902f4731c8c33ab16044ff3cebe645b5761dff33ea17d5a0f8149366-init/diff:/var/lib/docker/overlay2/af869e7618285c8997804a4288419c4c5fa15f99ab74bad8986dad5a5f302713/diff",
                "MergedDir": "/var/lib/docker/overlay2/7d6ff5f6902f4731c8c33ab16044ff3cebe645b5761dff33ea17d5a0f8149366/merged",
                "UpperDir": "/var/lib/docker/overlay2/7d6ff5f6902f4731c8c33ab16044ff3cebe645b5761dff33ea17d5a0f8149366/diff",
                "WorkDir": "/var/lib/docker/overlay2/7d6ff5f6902f4731c8c33ab16044ff3cebe645b5761dff33ea17d5a0f8149366/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "
]
```



##### 6.13通常容器	都是使用后台的方式运行的，需要进入容器，修改一些配置

```shell
#命令
#docker exce -it 容器ID /bin/sh

[root@localhost ~]# docker exec -it 9b /bin/sh
#结果
sh-4.4# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
```



		#docker attach 容器ID 
		正在进行的代码。。。。
	
		#docker exec #进入一个新的终端，可以在里面操作
		#docker attach #进入容器正在执行的终端，不会启动新的进程。

##### 6.14从容器内拷贝文件到主机

```shell
docker cp 容器ID:容器内路径 目的路径
[root@localhost ~]# docker cp 66:/luowei.python ./
[root@localhost ~]# ll
总用量 4
-rw-------. 1 root root 1241 1月  18 12:33 anaconda-ks.cfg
-rw-r--r--. 1 root root    0 1月  19 21:14 luowei.python
[root@localhost ~]# 


```

```shell
作业2：
docker run -it --rm tomcat:9.0
#我们之前启动的都是后台，停止容器之后，容器还是可以查到，docker run -it --rm tomcat,一般用来测试，用完自动删除。
#docker pull tomcat
#docker run -d -p 3355:8080 --name tomcat01 tomcat
```



```shell
作业3：部署es+kibana

##### es暴露的端口多

#es十分耗内存
#es的数据一般需要放置到安全目录挂载
#启动elasticsearch 
[root@localhost ~]#docker stats #查看内存状态
[root@localhost ~]# docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 \
 -e 'discovery.type=single-node' elasticsearch:7.6.2
[root@localhost ~]#docker stats #查看内存状态 
#发现启动了服务，内存消耗严重。
#es十分消耗内存 
#查看docker stats
 修改后
[root@localhost ~]# docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e 'discovery.type=single-node' \
 -e ES_JAVA_OPTS='-Xms256m -Xmx256m' elasticsearch:7.6.2    #通过限制容器内存上限。

[root@localhost ~]# curl localhost:9200
{
  "name" : "92fc36b0da30",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "q1claEP3TtSaf3FimGTp3Q",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
[root@localhost ~]#docker stats #查看内存状态
作业5：部署kibana

#可视化
   portainer(先用这个）
[root@localhost ~]#docker run -d -p 8088:9000 \
	--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true   portainer/portainer
   Rancher(CI/CD）在用
什么是portainer
  docker图形化界面管理工具，提供一个后台面板共我们操作

在笔记本上测试上管理界面 http://192.168.31.129:8088/#/init/admin #测试
思考问题：我们以后要部署项目，如果每次都要进入容器十分麻烦？我要是可以在容器外部提供webappps/*，自动同步到容器就好了。


```

#######################################################################################################

```shell
#作业1：
#用docker 部署服务
[root@localhost ~]#docker search --filter=STARS=3000 nginx
[root@localhost ~]# docker search --filter=STARS=3000 --no-trunc nginx
NAME      DESCRIPTION                STARS     OFFICIAL   AUTOMATED
nginx     Official build of Nginx.   14291     [OK]    
[root@localhost ~]#docker pull nginx   #下载nginx
[root@localhost ~]# docker run -d  --name nginx01 -p 3344:80 nginx
8c5a098ba14641fdc331617f82fd3e66a6c3a7b795e07c932a19a0abdbb0d0ca
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
8c5a098ba146   nginx     "/docker-entrypoint.…"   3 seconds ago    Up 2 seconds    0.0.0.0:3344->80/tcp   nginx01
[root@localhost ~]# curl localhost:3344   #访问成功

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>



<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>



<p><em>Thank you for using nginx.</em></p>

</body>
</html>
####################################################################
```

##### 如何提交一个自己的镜像

------

```shell
commit 镜像

docker commit 提交容器成为一个新的副本

#命令和git 类似

docker commit -m='提交的描述信息' -a='作者’ 容器ID 目标镜像名:[TAG]

#实战测试 做到实践和理论知识相结合。
[root@localhost ~]#docker run -it tomcat:9.0 /bin/bash
[root@localhost ~]#docker ps
[root@localhost ~]#docker commit -a='luowei' -m='add webapps appp' 11 tomcat02:1.0
[root@localhost ~]#docker ps
[root@localhost ~]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
tomcat02              1.0       d79405ae79b5   2 minutes ago   653MB
tomcat                9.0       040bdb29ab37   6 days ago      649MB
tomcat                latest    040bdb29ab37   6 days ago      649MB
nginx                 latest    f6d0b4767a6c   7 days ago      133MB
mysql                 latest    d4c3cafb11d5   7 days ago      545MB
centos                latest    300e315adb2f   6 weeks ago     209MB
portainer/portainer   latest    62771b0b9b09   6 months ago    79.1MB
elasticsearch         7.6.2     f29a1ee41030   9 months ago    791MB
hello-world           latest    bf756fb1ae65   12 months ago   13.3kB
```

##### Docker镜像原理

| ⒈是什么？                                                    |      |      |      |      |      |
| ------------------------------------------------------------ | ---- | ---- | ---- | ---- | ---- |
| 镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量以及配置文件等。 |      |      |      |      |      |
| 引用                                                         |      |      |      |      |      |
| UnionFs（联合文件系统）：union文件系统（UnionFs）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改   作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件下。 |      |      |      |      |      |
| Union文件系统是Docker镜像的基础，镜像可以通过分层来进行继承，基于基础镜像，可以制作各种具体的应用镜像。 |      |      |      |      |      |
| 特性：一次同时加载多个文件系统，但从外面看来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。 |      |      |      |      |      |
|                                                              |      |      |      |      |      |
|                                                              |      |      |      |      |      |

##### Docker镜像加载原理

| Docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFs |      |      |
| ------------------------------------------------------------ | ---- | ---- |
| Bootfs（boot-file system）主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs，这一层与我们典型的Linux/unix系统是一样的，包含boot加载器和内核，当boot加载完成之后整个内核就能在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。 |      |      |
| Rootfs（root-file system），在bootfs之上，包含的就是典型Linux系统中的/dev、/proc、/bin、/etc等标准目录和文件，rootfs就是各种不同操作系统的发行版，比如Ubuntu，Centos等等。 |      |      |
| 对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序就可以了，因为底层直接用宿主机的内核，自己只需要提供rootfs就可以了，因此可见，对于不用的Linux发行版，bootfs基本是一致的，而rootfs会有差别，因此不同的发行版可以公用bootfs。 |      |      |

##### Docker为什么采用分层结构呢？

　　--共享资源

　　多个镜像从相同的父镜像构建而来，那么宿主机只需在磁盘上保存一份父镜像，同时内存中也只需加载一份父镜像就可以为所有容器服务了，并且镜像的每一层都可以被共享。

##### Docker镜像特点

　　①Docker镜像只读

　　②当镜像实例为容器后，只有最外层是可写的。


















































