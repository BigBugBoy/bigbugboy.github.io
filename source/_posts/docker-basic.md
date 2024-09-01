---
title: Docker 基础入门
tags: docker
date: 2024-09-01 10:54:18
---





# Centos7 安装 docker

>本节安装说明适用于在腾讯云云服务器上安装 docker
>
>如果有条件科学上网，可以参考 docker 官网安装文档：[https://docs.docker.com/engine/install/centos/](https://gitee.com/link?target=https%3A%2F%2Fdocs.docker.com%2Fengine%2Finstall%2Fcentos%2F)

<br>

1. **添加腾讯云 yum 源**。命令执行后会在 `/etc/yum.repo.d`路径下出现一个新文件 `docker.ce.repo`

```bash
sudo yum-config-manager --add-repo=https://mirrors.cloud.tencent.com/docker-ce/linux/centos/docker-ce.repo
```

<br>

2. **查看已添加的 docker 软件源**

```bash
sudo yum list docker-ce
```

<br>

3. **yum 安装 docker**

```bash
sudo yum install -y docker-ce 
```

<br>

4. **运行 docker**

```bash
sudo systemctl start docker
```

<br>

5. **查看 docker 版本**

```bash
sudo docker version
```

<br>

6. **配置 docker 镜像源**

- 编辑或新建文件  `/etc/docker/daemon.json`，将如下内容复制到文件中并并保存。
- 执行命令后，重启 docker 即可，后面使用 `docker run` 下载镜像就会使用腾讯源了。

```json
{
   "registry-mirrors": [
   "https://mirror.ccs.tencentyun.com"
  ]
}
```

<br>

7. **管理 docker 服务**

- 启动服务 `systemctl start docker`
- 关闭服务 `systemctl stop docker`
- 重启服务 `systemctl restart docker`
- 查看状态 `systemctl status docker`
- 开机自启 `systemctl enable docker`





# Mac 安装 docker

Mac 安装 docker 非常方便，可以直接到 docker 官网下载最新版本的 docker desktop，然后点击安装。安装完成后，在启动台中找到 docker 图标，点击启动 docker 即可。随后就可以在控制台中使用 docker 指令，也可以在 docker desktop 中管理镜像和容器等。

docker官网：https://www.docker.com/





# xshell 和 xftp

NetSarang Computer,Inc.以过去10年免费提供强大的SSH和SFTP/FTP客户端而自豪。他们的免费许可证不仅是免费的价格，而且没有广告或其他剥削用户的方式。

他们认为，来自各种背景和环境的用户都应该能够访问功能强大、功能丰富的SSH和SFTP/FTP客户机。无论是学习、教学，还是仅仅是作为一种爱好的补充。更新：从 2022/02/16 开始，他们的免费许可证的标签限制已被删除。所有免费用户现在都可以通过下载最新版本来访问无限的标签。

免费版本的官方下载地址：https://www.xshell.com/zh/free-for-home-school/





# 仓库镜像容器的关系

- Docker 镜像（Image）：镜像是一个只读的模板，可以用来创建容器。
- Docker 容器（Container）：容器是镜像的运行实例，它是一个独立的环境，可以在这个环境中运行应用程序，一个镜像可以创建多个容器。
- Docker 仓库（Repository）：Docker 仓库是用来存储 Docker 镜像的地方。仓库可以分为本地仓库和远程仓库， 比如 DockerHub 是官方的镜像仓库，同时也是一个远程仓库，我们可以在这里下载各种镜像，也可以将自己的镜像上传到这里。

>补充：仓库镜像容器的概念非常重要。为了更好的理解三者的关系，可以这样来理解：镜像就相当于一个QQ安装包，这个安装包可以在电脑上安装成为QQ程序（相当于容器），也可以继续在其他电脑上安装另一个QQ程序（甚至可以在一个电脑上登陆多个QQ），QQ安装包从应用市场（仓库）下载的。





# 镜像管理的常用指令

**从远程仓库下载镜像**

~~~bash
docker pull <image>
~~~

- 直接使用 `docker pull image` 默认下载最新版本的镜像，即此时的镜像版本标签为 `latest`

- 如果下载指定版本的镜像，则可以加上版本号 `docker pull image-name:tag`
- 该指令等价于 `docker image pull <image>`

<br>



**查看本地镜像列表**

~~~bash
docker images
~~~

- 等价于 `docker image list`，等价于 `docker iamge ls`

<br>



**删除镜像**

~~~bash
docker rmi 镜像id 或者 镜像名:tag
~~~

- 等价于 `docker image remove`， 等价于 `docker image rm`
- 如果想要强制删除镜像，可以使用删除指令时增加选项 `-f`
- 支持同时删除多个镜像

<br>



**基于 dockerfile 构建镜像**

```bash
docker build -t bigbugbou/hello-docker:v1 -f Dockerfile .
```

- 使用 Dockerfile 构建镜像是主流的推荐方式，我们在后面会详细介绍。
- 参数 `-t` 指定待构建镜像的镜像名和版本
- 参数 `-f` 指定Dockerfile文件的路径，默认是当前路径下的 `Dockerfile` 文件
- 最后的 `.` 表示当前路径下的 context

<br>



**基于容器创建镜像**

```bash
docker commit <容器id> bigbugboy/my-nginx:v1
```

- 等价于 `docker image commit`
- commit 时可以通过指定容器id 或者容器名来创建镜像

<br>



**把镜像打包为文件**

~~~bash
docker save nginx -o /root/nginx.zip
~~~

- 等价于 `docker iamge save`
- 参数 `-o` 表示打包后的文件存放路径

<br>



**从文件解压得到镜像**

```bash
docker load -i /root/nginx.zip
```

- 等价于 `docker image load`
- 参数 `-i` 表示被解压文件的路径

<br>



**上传镜像到远程仓库**

```bash
docker push bigbugboy/my-nginx:v1
```

- 等价于 `docker iamge push`

<br>



# 容器管理的常用指令

**创建容器（仅创建，不运行）**

```
docker create <image>
```

<br>

**启动容器**

```
docker start <container>
```

<br>

**创建并运行容器**

```
docker run <image>
```

<br>

**查看容器列表**

```
docker ps
```

- 等价于 `docker container ls`
- 该命令只列出正在运行的容器，如果想要列出所有容器，加上参数 `-a`

<br>

**停止容器**

```
docker stop <container>
```

<br>

**重启容器**

```
docker restart <container>
```

<br>

**删除容器**

```
docker rm <container>
```

- 等价于 `docker container rm`
- 如果要删除正在运行的容器，需要强制删除，可以加上参数 `-f`

<br>

**进入容器**

```
docker exec -it <container> bash
```

- 以交互模式进入容器

<br>



**查看容器日志**

```
docker logs <container>
```

<br>





# 运行容器

运行容器一般指的是 `docker run` 命令，这个命令非常重要，主要体现在它可以配合很多参数实现不同的功能。

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# OPTIONS:
# -d: 后台运行容器
# -i: 以交互模式运行容器，通常与 -t 同时使用
# -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
# --name: 为容器指定一个名字
# --rm: 容器退出后自动删除容器文件
# -p: 端口映射，格式为：主机(宿主)端口:容器端口
# -v: 挂在数据卷
# -e: 为容器设置环境变量
# -w: 指定容器的工作目录
# --network: 指定容器的网络连接类型
# --link: 把一个容器的网络和另一个容器联通（单向）
```

创建一个 nginx 容器并指定容器名称，后台运行，环境变量，工作目录，退出自动删除

```bash
docker run -d --rm -e XYZ=hello -w /root --name my-nginx nginx
```

<br>



# 端口映射

Docker 端口映射是一种常见的 Docker 容器网络配置，用于将容器内部的端口映射到宿主机上的端口，从而可以通过宿主机的端口访问容器内部的服务。

在 Docker 中，端口映射可以通过 `-p` 或 `--publish` 标志来实现。以下是 Docker 端口映射的基本用法：

~~~bash
docker run -d -p [host_port]:[container_port] [image_name]
~~~

- `-d`: 在后台运行容器。
- `-p [host_port]:[container_port]`: 这里 `[host_port]` 是宿主机上的端口，`[container_port]` 是容器内部的端口。通过这种映射，宿主机的 `[host_port]` 端口将会被映射到容器内部的 `[container_port]` 端口上。
- `[image_name]`: 要运行的 Docker 镜像名称。



例如，如果要将容器内的 80 端口映射到宿主机的 8080 端口，可以这样运行容器：

~~~bash
docker run -d -p 8080:80 nginx
~~~



这将会启动一个 NGINX 容器，并将容器内的 80 端口映射到宿主机的 8080 端口上。现在，可以通过访问 `http://localhost:8080` 来访问 NGINX 服务。

>注意：宿主机端口和容器端口可以不一样，但是宿主机端口必须是未被占用的端口。







# 数据卷

在 Docker 中，数据卷挂载是一种常见的技术，用于将宿主机上的目录或文件挂载到容器内部，从而实现容器与宿主机之间的数据共享和持久化存储。数据卷可以绕过容器文件系统，使数据在容器删除后仍然存在。

在 Docker 中，数据卷可以通过两种方式进行创建和使用：一种是使用 `docker volume create` 命令创建数据卷，另一种是直接在容器运行时通过 `-v` 参数创建数据卷并挂载到容器中。

>**`docker volume create` 方式**：
>
>- **创建数据卷**：使用 `docker volume create` 命令可以创建一个独立的数据卷。
>- **命名自定义**：可以为数据卷指定自定义名称，以便于识别和管理。
>- **独立存在**：创建的数据卷是独立于容器的，可以在多个容器之间共享使用。
>- **持久化存储**：数据卷中的数据在容器删除后仍然存在，可以实现数据的持久化存储。
>
>**容器运行时创建数据卷**：
>
>- **即时创建**：可以在容器运行时通过 `-v` 参数创建数据卷并挂载到容器中，而无需提前创建数据卷。
>- **临时使用**：这种方式创建的数据卷通常是临时性的，适合一次性使用，不需要为数据卷单独命名和管理。
>- **持久化存储**：临时创建的数据卷中的数据在容器删除后仍然存在，除非手动删除。





## 容器运行时创建

```bash
docker run -v [host_path]:[container_path] [image_name]
```

- `-v [host_path]:[container_path]`: 这里 `[host_path]` 是宿主机上的路径，`[container_path]` 是容器内部的路径。这将会把宿主机上的 `[host_path]` 目录挂载到容器内的 `[container_path]` 目录上。
- `[image_name]`: 要运行的 Docker 镜像名称。

例如，如果要将宿主机的 `/opt/data` 目录挂载到容器内的 `/data` 目录，可以这样运行容器：

~~~bash
docker run -v /opt/data:/data [image_name]
~~~



**查看宿主机上挂载卷地址**

- 终端输入下面的命令，就可以看到了。

~~~bash
docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
~~~







## docker volume create 方式

**创建一个数据卷**

```
docker volume create <volume-nme>
```

<br>

**查看数据卷**

```
docker volume ls
```

<br>

**查看数据卷详细信息**

```
docker volume inspect <vlume-name>
```

<br>

**删除数据卷**

```
docker volume rm <volume-name>
```

<br>

**将数据卷挂载到容器**

~~~bash
docekr run -d -v mydata:/data <image>
~~~

- 这样会把名字为 `mydata` 的数据卷挂载到容器中的 `/data` 目录上。







# docker 网络

docker 默认会创建三个网络，分别是 bridge、host、none。

- bridge：桥接网络，docke r默认使用的网络模式，使用 `docker run` 命令创建容器时如果不指定网络模式，那么就会使用 bridge 模式。
- host：主机网络，使用宿主机的网络，容器将不会获得一个独立的网络命名空间，配置和宿主机共享，容器将不会隔离宿主机网络，使用宿主机的IP和端口。
- none：无网络、禁用网络，容器拥有自己的网络命名空间，但是并不为容器进行任何网络配置，这个网络模式的容器只适合于只进行数据处理，没有任何网络的应用场景。



**列出网络**

```
docker network ls
```

<br>

**查看网络信息**

```
docker network inspect <network>
```

<br>

**创建网络**

```
docker network create <network>
```

- 创建自定义网络是，可以使用参数 `--driver` 指定创建的网络属于什么类型，默认是 `bridge`

<br>

**删除网络**

```
docker network rm <network>
```

<br>

**运行容器时指定挂载网络**

~~~bash
docker run --network bridge --name my-nginx nginx
~~~

- 不指定 `--network` 即默认挂载在 `bridge`上，即挂载于 `docker0`上。





# docker0网卡

docker安装并启动服务后，会在宿主机中添加一个虚拟网卡， `docker0` 网卡。这个 `docker0` 就是 `bridge` 网络。

>使用 `ifconfig` 或 `ip addr` 查看网卡信息



默认情况下通过 `docker run` 运行的容器都挂在 `docker0` 网卡上，即他们之际是在一个 ip 段上，网络互通。换句话说，换句话说在这些容器内可以痛通过 ip 互联。并且以为 `docker0` 在其中起到了连接容器和宿主机的作用，容器和宿主机支架也可以互联。

启动两个 nginx 容器

~~~bash
docker -d --name nginx1 nginx
docker -d --name nginx2 nginx

docker ps
CONTAINER ID   IMAGE                          COMMAND                   CREATED          STATUS                    PORTS                  NAMES
c2c1607b684d   nginx                          "/docker-entrypoint.…"   27 seconds ago   Up 26 seconds             80/tcp                 nginx2
07b78d04956b   nginx                          "/docker-entrypoint.…"   43 seconds ago   Up 40 seconds             80/tcp                 nginx1           80/tcp                 nginx1
~~~

通过 `ip add` 查看容器 `docker0` 网卡所在的 ip 段是 `172.17.0.1/16`

查看 `docker0`网络上挂载的容器，可以看到两个容器的 ip

~~~bash
docker network inspect bridge
~~~



测试 宿主机访问容器 ip，容器访问宿主机器ip，容器之间互访ip

- 使用 `ip add` 查看IP地址，使用`ping` 测试是否可以访问

~~~bash
[root@VM-12-14-centos ~]# ping 172.17.0.3 -c 2
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.039 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.042 ms

--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.039/0.040/0.042/0.006 ms
~~~





# 容器内安装网络工具



Apt-get 换源

>参考：
>
>https://mirrors.tencent.com/help/debian.html
>
>https://mirrors.tuna.tsinghua.edu.cn/help/debian/

- 因为是在阿里云服务器上，直接使用内网地址（http://mirrors.tencentyun.com）
- 修改 `/etc/apt/sources.list.d/debian.sources` 为如下内容

~~~bash
Types: deb
URIs: http://mirrors.tencentyun.com/debian-security
Suites: bookworm-security
Components: main
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb
URIs: http://mirrors.tencentyun.com/debian
Suites: bookworm bookworm-updates bookworm-backports
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
~~~



容器内安装工具

- 使用 `ip` 命令，需要安装 `iproute2`
- 使用 `ping` 命令，需要安装 `iputils-ping`
- 使用 `ifconfig` 命令，需要安装`net-tools`



~~~bash
apt-get clean all
apt-get update
apt-get install -y iproute2
apt-get install -y iputils-ping
~~~





# 虚拟网卡对 veth pair

顾名思义，veth-pair 就是一对的虚拟设备接口，它都是成对出现的。一端连着协议栈，一端彼此相连着

Docker 创建一个容器的时候，会执行如下操作：

 • 创建一对veth pair，分别放到本地主机和容器中。
 • 本地主机一端桥接到默认的 docker0 或指定网桥上，并具有一个唯一的名字，如 vethced5325；
 • 容器一端放到新容器中，并修改名字作为 eth0，这个网卡/接口只在容器的名字空间可见；
 • 从网桥可用地址段中（也就是与该bridge对应的network）获取一个空闲地址分配给容器的 eth0，并配置默认路由到桥接网卡 vethced5325。

完成这些之后，容器就可以使用 eth0 虚拟网卡来连接其他容器和其他网络。如果不指定--network，创建的容器默认都会挂到 docker0 上，使用本地主机上 docker0 接口的 IP 作为所有容器的默认网关。

<img src="https://raw.githubusercontent.com/bigbugboy/pic/main/img/docker0.png?s" alt="docker0" style="zoom:67%;" />





# docker可视化工具



Portainer 是一款 Docker 可视化管理工具，可让您轻松构建和管理 Docker、Docker Swarm、Kubernetes 和 Azure ACI 中的容器。 Portainer 将管理容器的复杂性隐藏在易于使用的 UI 后面。

官方地址：https://www.portainer.io/



**安装使用**

- 详看文档：https://docs.portainer.io/start/install-ce/server

- linux安装 portainer 

~~~bash
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:2.21.0
~~~

