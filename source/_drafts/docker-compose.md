---
title: docker-compose
tags:
---

# docker compose 前戏

在详细介绍docker compose 之前，我们来聊聊为什么需要它，在什么场景下使用它，使用它帮我们解决了什么痛点。

需求：使用 docker 部署一个 flask 服务，flask服务需要一个配套的 redis 服务。

**app.py** 简单的web项目，访问一次就会在 redis 中增加一次浏览次数。

~~~python
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
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
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





# docker-compose 和 docker compose

# docker-compose 快速上手

# docker-compose 常用命令行指令

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