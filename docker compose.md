### Docker Compose

#### 简介

Docker Compose 简单高效的管理容器 定义运行多个容器

官方介绍

1.yaml 配置文件

2.docker compose有哪些命令

作用：批量容器编排

> 狂神的理解

1.Compose 是docker官方的开源项目，需要安装

2.Dockerfile 让程序在任何地方运行，web服务，redis,mysql多个容器

Compose  yaml文件例子

```yaml
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

#### 安装Compose

```shell
官方文档操作步骤 帮助连接：https://docs.docker.com/compose/install/

[root@localhost local]# sudo curl -L "https://github.com/docker/compose/releases/download/1.28.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose   #国外网站速度慢

[root@localhost local]# sudo chmod +x /usr/local/bin/docker-compose
[root@localhost local]# ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
[root@localhost local]# docker-compose --version
```

#### Compose :重要的概念

1. 服务services,容器，应用（web,redis,mysql)
2. 项目project,一组关联的容器。

集群的方式部署容器，4台阿里云服务器。2核4G

官方操作  **https://docs.docker.com/compose/gettingstarted/**

1.准备工作

>1.yum install python-pip  #pip 是的包管理工具
>
>2.yum -y install epel-release    #

2 .创建项目目录

```shell
1.mkdir composetest
2.cd composetest
```



3.在项目目录中，创建一个文件app.py ，内容如下

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```



4.在项目目录中创建另一个名为requiremnets.txt的文件 内容

> flask
>
> redis

5.在项目中创建名为Dockerfile的文件

```shell
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["python", "app.py"]
```

6.docker-compose.yaml 文件（定义整个服务需要的环境，web,redis) 完整的上线服务器

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

7.启动compose 项目（docker-compose up)

