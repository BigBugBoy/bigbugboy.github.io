---
title: docker-compose
tags:
---

Docker Compose 是一个定义和运行多容器应用程序的工具。Compose 简化了整个应用程序的控制，让你能够轻松地在一个简单易懂的 YAML 配置文件中管理服务、网络和卷。只需使用一个命令即可从配置文件中创建和启动所有服务。

Compose 适用于所有环境；生产、准备、开发、测试以及 CI 工作流。它还具有用于管理应用程序整个生命周期的命令：

- 启动、停止和重建服务
- 查看正在运行的服务的状态
- 流式输出正在运行的服务的日志
- 在服务上运行一次性命令

**Compose 的主要优点**，简化容器化应用程序的开发、部署和管理：

- 简化控制：Docker Compose 允许您在单个 **YAML** 文件中定义和管理多容器应用程序。这简化了编排和协调各种服务的复杂任务，使管理和复制应用程序环境变得更加容易。
- 高效协作：Docker Compose 配置文件易于共享，促进开发人员、运营团队和其他利益相关者之间的协作。这种协作方法可实现更顺畅的工作流程、更快的问题解决速度并提高整体效率。
- 快速应用程序开发：Compose 缓存用于创建容器的配置。当您重新启动未更改的服务时，Compose 会重新使用现有容器。重复使用容器意味着您可以非常快速地更改环境。
- 跨环境的可移植性：Compose 支持 Compose 文件中的变量。您可以使用这些变量针对不同的环境或不同的用户自定义您的组合。
- 广泛的社区和支持：Docker Compose 受益于充满活力的社区，这意味着丰富的资源、教程和支持。这个社区驱动的生态系统有助于 Docker Compose 的持续改进，并帮助用户有效地解决问题。

**Docker Compose 的常见使用场景**

- 开发环境：在开发软件时，在隔离环境中运行应用程序并与之交互的能力至关重要。Compose 命令行工具可用于创建环境并与之交互。

- 自动化测试环境：任何持续部署或持续集成流程的一个重要部分是自动化测试套件。

```console
$ docker compose up -d
$ ./run_tests
$ docker compose down
```

- 单主机部署：Compose 传统上一直专注于开发和测试工作流程，但随着每个版本的发布，我们都在更多面向生产的功能方面取得进展。

<br>



# docker compose 前戏

在详细介绍docker compose 之前，我们来聊聊为什么需要它，在什么场景下使用它，使用它帮我们解决了什么痛点。

需求：使用 docker 部署一个 flask 服务，flask服务需要一个配套的 redis 服务。

**app.py** 简单的web项目，访问一次就会在 redis 中增加一次浏览次数。

~~~python
import time

import redis
from flask import Flask


app = Flask(__name__)
# 如果这里使用的是 redis 的ip则更麻烦
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
    return f'Hello World! I have been seen {count} times.\n'
~~~

对于这个项目，使用 docker 来部署需要拆分为两个容器，一个是web容器，一个是 redis 容器。其中，web 容器依赖 redis 容器，且在 web 容器中需要通过字符串 `redis` 这个名字找到 redis 服务的，而不是通过写死的 IP 地址。

基于这个情况，我们需要分步骤依次实现。

第一步：创建一个 redis 容器

~~~bash
docker run -d -P --name redis redis
~~~

<br>

第二步：准备 web 容器的 Dockerfile，用来构建 web 容器需要的镜像

~~~dockerfile
FROM python:3.10-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
EXPOSE 5000
COPY . .
CMD ["flask", "run", "--debug"]
~~~

其中项目的依赖文件 requirements.txt

~~~tex
flask
redis
~~~

<br>

第三步：构建 web 镜像

~~~bash
docker build -t flask-app:v1 -f Dockerfile --no-cache .
~~~

<br>

第四步：运行 web 容器，并和 redis 容器联通

~~~bash
docker run -d --name web -p 5000:5000 --link redis flask-app:v1
~~~

<br>

第五步：访问项目

~~~bash
curl http://127.0.0.1:5000/
~~~

<br>



总结：这个需求很简单，思路也很简单，但是实现起来很繁琐。需要手动运行两个容器，并且容器之间有依赖关系和关联关系。

<br>



# docker compose 快速上手

>本节的主要目的是让大家快速上手 docker compose，具体细节不用在意，后面会详细介绍。我们先从整体上认识 docker compose 的基本使用流程。

基于上节 flask-app项目，我们使用 compose的方式。第一步先准备一个 compose 文件。

**compose.yaml**

~~~yaml
services:
  redis:
    image: redis
  web:
    build: .
    ports:
      - "5000:5000"
~~~

**Compose 构建镜像**

~~~bash
docker compose build --no-cache
~~~

**Compose 运行服务**

~~~bash
docker compose up -d
~~~

**查看运行的compose项目**

~~~bash
docker compose ls
~~~

查看compose项目包含的容器

~~~bash
docker compose ps
~~~

启动、停止、清除项目

~~~bash
docker compose up
docker compose stop [service]
docker compose start [service]
docker compose restart [service]
docker compose down
~~~

<br>



# docker compose 常用命令行指令

| 命令    | 描述                                              |
| ------- | ------------------------------------------------- |
| build   | 构建或重新构建服务                                |
| cp      | 在服务容器和本地文件系统之间复制文件/文件夹       |
| down    | 停止并删除容器、网络                              |
| exec    | 在运行中的容器中执行命令                          |
| images  | 列出由创建的容器使用的镜像                        |
| kill    | 强制停止服务容器                                  |
| logs    | 查看容器的输出                                    |
| ls      | 列出正在运行的 Compose 项目                       |
| ps      | 列出容器                                          |
| restart | 重新启动服务容器                                  |
| rm      | 移除已停止的服务容器                              |
| run     | 在一个服务上运行一次性命令                        |
| scale   | 扩展服务                                          |
| start   | 启动服务                                          |
| stats   | 显示容器资源使用统计数据的实时流                  |
| stop    | 停止服务                                          |
| top     | 显示正在运行的进程                                |
| up      | 创建并启动容器                                    |
| version | 显示 Docker Compose 版本信息                      |
| watch   | 监视服务构建上下文，当文件更新时重新构建/刷新容器 |

<br>



# docker compose 历史

Compose是一个容器编排工具。其实，Fig 项目在开发者面前第一次提出了“容器编排”（Container Orchestration）的概念。 后来，Fig 项目被收购后Docker公司并改名为 Compose。

Docker Compose 的第一版于 2014 年发布。它用 Python 编写，并使用 `docker-compose` 命令调用。

Docker Compose 的第二版于 2020 年发布，用 Go 编写，并使用 `docker compose` 命令调用。

![](https://docs.docker.net.cn/compose/images/v1-versus-v2.png)



对于初学者来说，如果你使用的是最新版本的 docker，可以直接使用 `docker compose` 这个命令来使用 compose工具；对于老版本的 docker 用户，因为 compose没有集成到docker中，你需要单独安装 docker-compose 工具，然后使用 `docker-compose` 命令来使用 compose 工具。

在 linux 上单独安装 docker-compose 

~~~bash
curl -SL https://github.com/docker/compose/releases/download/v2.29.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
~~~

对二进制文件应用可执行权限

~~~bash
chmod +x /usr/local/bin/docker-compose
~~~

测试

~~~bash
docker-compose
~~~

>如果使用的是 docker desktop，则无需安装直接使用 `docker-compose`。另外，`docker compose` 和 `docker-compose` 的命令和用法完全一致（比如：`docker compose up -d` 等价于 `docker-compose up -d`），使用其中一种即可。 



**注意：**

- 如果使用的是 `docker-compose `命令，一般配套使用 `docker-compose.yaml`文件。

- 如果使用的是 `docker compose` 命令，一般配套使用 `compose.yaml`文件，但是它也支持使用 `docker-compose.yaml`文件。如果同时存在，则优先使用 `compose.yaml`文件。

<br>



# compose 文件的结构

Compose 规范是 Compose 文件格式的最新版本，也是推荐版本。它可以帮助你定义一个 Compose 文件，用于配置 Docker 应用程序的服务、网络、卷等。

Compose 文件顶层有4个常用配置项：`name`、`services`、`networks`、`volumes`

**name**

- 顶层 `name` 属性由 Compose 规范定义，作为项目名称。

**services**

- Compose 文件必须声明一个`services`顶级元素作为映射，其键是服务名称的字符串表示形式，其值是服务定义。每个服务定义运行时约束和要求以运行其容器。
- 可以有多个服务，一个服务就是一个容器。

**networks**

- 顶级 `networks` 使你能够配置可在多个服务之间相互通信的网络。默认情况下，Compose 为您的应用程序设置单个网络。每个服务的容器都加入默认网络，并且可以与该网络上的其他容器通信，并且可以通过服务的名称发现。

- 要跨多个服务使用网络，必须通过在 `services` 顶级元素中使用 [networks](https://docs.docker.net.cn/compose/compose-file/05-services/) 属性来显式授予每个服务访问权限。

**volumes**

- 顶级 `volumes` 使你能够配置可在多个服务之间重复使用的命名卷。要在多个服务之间使用卷，必须使用 `services` 顶级元素内的 [volumes](https://docs.docker.net.cn/compose/compose-file/05-services/#volumes) 属性显式地授予每个服务访问权限。



**compose.yml**

~~~yaml
name: myapp

services:
  redis:
    image: redis
    networks:
      - my-network
  web:
    build: .
    ports:
      - "5000:5000"
    networks:
      - my-network
    volumes:
      - my-data:/etc/data

networks:
  my-network:

volumes:
  my-data:
~~~

<br>



# compose 构建镜像



#  compose 映射端口和环境变量

# compose 设置项目名称和容器名称 

# compose 文件指令之links和depends_on

# compose 文件指令之networks

# ompose 文件指令之volumes

# compose 文件指令之 include

# compose 文件指令之 watch

# compose 构建WordPress博客系统

# compose 构建项目Django项目

