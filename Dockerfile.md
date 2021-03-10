## Dockerfile

------

##### Dockerfile的指令

------



```shell
#FROM        #基础镜像，一切从这里构建，
#MAINTAINER  #镜像是谁写的，姓名+邮箱
#RUN         #镜像构建的时候需要运行的命令
#ADD         #格式：ADD src dest 该命令将复制指定本地目录中的文件到容器中的dest中，src可以是是一个绝对路径，也可以是一个URL或一个tar文件，tar文件会自动解压为目录。
#WORKDIR     #格式： WORKDIR /path  切换到指定目录
#EXPOSE      #暴露的端口
#ENV         #EVN key value 。用于指定环境变量
#COPY        #格式为：COPY src desc
#ENTRYPOINT  #ENTRYPOINT ["executable","param1","param2"]和ENTRYPOINT command param1,param2 会在shell中执行。
#VOLUME      #格式为 VOLUME ["/data"]作用是创建在本地主机或其他容器可以挂载的数据卷，用来存放数据。
#ONBUILD
```

#####   实战测试

------

`创建一个自己的dockerfile`

```shell
#编写dockerfile的文件
[root@localhost docker-test-volume]# cat dockerfile
FROM centos
MAINTAINER luowei<2570473902@qq.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD echo '---end---'
CMD /bin/bash
#通过dockerfile构建镜像
[root@localhost docker-test-volume]#docker build -f /root/docker-test-volume/docker -t mycentos .
```

```shell
#区别CMD 和ENTRYPOINT
[root@localhost docker-test-volume]# cat dockerfile2
FROM centos
CMD ["ls","/"]
[root@localhost docker-test-volume]# cat dockerfile3
FROM centos
ENTRYPOINT ["ls","/"]
[root@localhost docker-test-volume]# docker images
REPOSITORY            TAG       IMAGE ID       CREATED             SIZE
entrypoint            latest    c5e7138ccdb6   About an hour ago   209MB
cmdtest               latest    b8e9ef73fe2b   About an hour ago   209MB

[root@localhost docker-test-volume]# docker run c5e7138ccdb6   #entrypoint
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
[root@localhost docker-test-volume]# docker run b8e9ef73fe2b   #CMD
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
#目前是没有什么区别的
[root@localhost docker-test-volume]# docker run b8e9ef73fe2b -l #ls / 被-l给替换掉了 所以执行宝座
docker: Error response from daemon: OCI runtime create failed: container_linux.go:370: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
#cmd的角色定位就是默认，如果你不额外指定，那么就执行cmd的命令，否则呢？只要你指定了，那么就不会执行cmd，也就是cmd会被覆盖。run后面制定了-l这个参数，覆盖ls / 命令，所以-l 在linux系统中执行汇报报错。
#注意如果run 后面跟的是一条完整的命令，也不会报错

#entrypoint
[root@localhost docker-test-volume]# docker run b8e9ef73fe2b ls -l
total 0
lrwxrwxrwx.   1 root root   7 Nov  3 15:22 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Jan 20 17:42 dev
drwxr-xr-x.   1 root root  66 Jan 20 17:42 etc
drwxr-xr-x.   2 root root   6 Nov  3 15:22 home
lrwxrwxrwx.   1 root root   7 Nov  3 15:22 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3 15:22 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4 17:37 lost+found
drwxr-xr-x.   2 root root   6 Nov  3 15:22 media
drwxr-xr-x.   2 root root   6 Nov  3 15:22 mnt
drwxr-xr-x.   2 root root   6 Nov  3 15:22 opt
dr-xr-xr-x. 125 root root   0 Jan 20 17:42 proc
dr-xr-x---.   2 root root 162 Dec  4 17:37 root
drwxr-xr-x.  11 root root 163 Dec  4 17:37 run
lrwxrwxrwx.   1 root root   8 Nov  3 15:22 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 Nov  3 15:22 srv
dr-xr-xr-x.  13 root root   0 Jan 18 12:09 sys
drwxrwxrwt.   7 root root 145 Dec  4 17:37 tmp
drwxr-xr-x.  12 root root 144 Dec  4 17:37 usr
drwxr-xr-x.  20 root root 262 Dec  4 17:37 var

[root@localhost docker-test-volume]# docker run c5e7138ccdb6 -l     #ENTRYPOINT #run 后面不用跟完整的命令就可以执行。
total 0
lrwxrwxrwx.   1 root root   7 Nov  3 15:22 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Jan 20 17:36 dev
drwxr-xr-x.   1 root root  66 Jan 20 17:36 etc
drwxr-xr-x.   2 root root   6 Nov  3 15:22 home
lrwxrwxrwx.   1 root root   7 Nov  3 15:22 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3 15:22 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4 17:37 lost+found
drwxr-xr-x.   2 root root   6 Nov  3 15:22 media
drwxr-xr-x.   2 root root   6 Nov  3 15:22 mnt
drwxr-xr-x.   2 root root   6 Nov  3 15:22 opt
dr-xr-xr-x. 126 root root   0 Jan 20 17:36 proc
dr-xr-x---.   2 root root 162 Dec  4 17:37 root
drwxr-xr-x.  11 root root 163 Dec  4 17:37 run
lrwxrwxrwx.   1 root root   8 Nov  3 15:22 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 Nov  3 15:22 srv
dr-xr-xr-x.  13 root root   0 Jan 18 12:09 sys
drwxrwxrwt.   7 root root 145 Dec  4 17:37 tmp
drwxr-xr-x.  12 root root 144 Dec  4 17:37 usr
drwxr-xr-x.  20 root root 262 Dec  4 17:37 var
#ENTRYPOINT与cmd有区别。ENTRYPOINT如果run命令后面有东西，那么后面的全部都会作为entrypoint的参数，如果run后面没有额外的东西，但是cmd有，那么cmd的全部内容会作为entrypoint的参数，这同时是cmd的第二种用法。当然如果要在run里面覆盖，也是有办法的，使用--entrypoint即可。
FROM centos
CMD ["p in cmd"]
ENTRYPOINT ["echo"]
```

#### 实战创建Tomcat 镜像

##### 1.准备tomcat和jdk的压缩包。

![image-20210121222624976](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210121222624976.png)

##### 2.编写dockerfile文件

官方命令Dockerfile,build会自动寻找这个文件，就不需要在build时，用-f指定dockerfile文件在哪个路径

```shell
[root@localhost tomcat]# cat Dockerfile 
FROM centos
ADD jdk-13.0.2_linux-x64_bin.tar.gz  /usr/local
ADD apache-tomcat-9.0.38.tar.gz    /usr/local
RUN yum -y install vim net-tools
ENV MYPATH /usr/local
WORKDIR $MYPATH
ENV JAVA_HOME /usr/local/jdk-13.0.2
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.38
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.38
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
EXPOSE 8080
ENTRYPOINT  /usr/local/apache-tomcat-9.0.38/bin/startup.sh && tail -F /url/local/apache-tomcat-9.0.38/logs/catalina.out

```

##### 3.构建镜像

```shell
[root@localhost tomcat]# docker build -t mytomcat .
```

##### 4.运行tomcat容器

```shell
[root@localhost tomcatlogs]# docker run -d -p 9090:8080 --name mytomcat2 -v /root/tomcat/test:/usr/local/apache-tomcat-9.0.38/webapps/test -v /root/tomcat/tomcatlogs:/usr/local/apache-tomcat-9.0.38/logs  mytomcat
```

##### 5.访问测试

```
[root@localhost tomcatlogs]#curl localhost:9090
```

```shell
[root@localhost test]# pwd
/root/tomcat/test  #对应着容器中/usr/local/apache-tomcat-9.0.38/webapps/test/
[root@localhost test]# cat index.jsp      #同步index.jsp到容器/usr/local/apache-tomcat-9.0.38/webapps/test/目录下
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html> 
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
<p>
   今天的日期是: <%= (new java.util.Date()).toLocaleString()%>
</p>s
</body> 
</html> 
[root@localhost test]# docker restart mytomcat2
[root@localhost test]# curl localhost:9090/test

```

##### 阿里云镜像服务器上

------

1.登陆阿里云

2.找到容器镜像服务

3.创建命名空间

![image-20210122001048785](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122001048785.png)

4.创建镜像仓库

![image-20210122001308778](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122001308778.png)

5.阿里云上操作步骤

![image-20210122001506759](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122001506759.png)

6.上传镜像到阿里云上

```shell
#1. 登录阿里云Docker Registry
[root@localhost docker]# docker login --username=luowei5aly registry.cn-qingdao.aliyuncs.com
#23. 将镜像推送到Registry
$ sudo docker login --username=luowei5aly registry.cn-qingdao.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-qingdao.aliyuncs.com/luowei/my:[镜像版本号]
$ sudo docker push registry.cn-qingdao.aliyuncs.com/luowei/my:[镜像版本号]
[root@localhost docker]# docker tag decaf3  registry.cn-qingdao.aliyuncs.com/luowei/my:v1
[root@localhost docker]# docker push registry.cn-qingdao.aliyuncs.com/luowei/my:v1
The push refers to repository [registry.cn-qingdao.aliyuncs.com/luowei/my]
09a2dd35bee8: Pushed 
11db45979bc9: Pushed 
1c77726c72c7: Pushing [===============================================>   ]  304.4MB/323MB
2653d992f4ef: Pushed 
```

#### 小结

![image-20210122002729035](C:\Users\wEI\AppData\Roaming\Typora\typora-user-images\image-20210122002729035.png)