# docker 集群-Swarm

参考<https://yeasy.gitbooks.io/docker_practice/swarm_mode/deploy.html>

## 环境：

### 安装docker-machine

用于创建服务节点。

参考<https://yeasy.gitbooks.io/docker_practice/machine/install.html>

github下载地址：https://github.com/docker/machine/releases

```sh
$ sudo curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine
$ sudo chmod +x /usr/local/bin/docker-machine
```

建议翻墙下载，时间久了会超时。curl和迅雷下的都很慢。

完成后，查看版本信息。

```bash
$ docker-machine -v
docker-machine version 0.13.0, build 9ba6da9
```



### 安装docker-compose

docker-compose可以使用 `docker-compose.yml` 一次配置、启动多个容器，这里我们也可以用它配置、启动多个服务。

安装参照<https://yeasy.gitbooks.io/docker_practice/compose/install.html>

### 创建本地主机

#### Virtualbox 驱动

使用 `virtualbox` 类型的驱动，创建一台 Docker 主机，命名为 test。

```bash
$ docker-machine create -d virtualbox test
```

这里会去下载一个镜像，boot2docker.iso，也不小，建议翻墙。这里可以使用IDM，32线程

你也可以在创建时加上如下参数，来配置主机或者主机上的 Docker。

`--engine-opt dns=114.114.114.114` 配置 Docker 的默认 DNS

`--engine-registry-mirror https://registry.docker-cn.com` 配置 Docker 的仓库镜像

`--virtualbox-memory 2048` 配置主机内存

`--virtualbox-cpu-count 2` 配置主机 CPU

更多参数请使用 `docker-machine create --driver virtualbox --help` 命令查看。

### 使用

创建好主机之后，查看主机

```bash
$ docker-machine ls

NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER       ERRORS
test      -        virtualbox   Running   tcp://192.168.99.187:2376           v17.10.0-ce
```

创建主机成功后，可以通过 `env` 命令来让后续操作对象都是目标主机。

```bash
$ docker-machine env test
```

后续根据提示在命令行输入命令之后就可以操作 test 主机。

也可以通过 `SSH` 登录到主机。

```bash
$ docker-machine ssh test

docker@test:~$ docker --version
Docker version 17.10.0-ce, build f4ffd25
```

连接到主机之后你就可以在其上使用 Docker 了。



## 集群

参考<https://yeasy.gitbooks.io/docker_practice/swarm_mode/create.html>

阅读 [基本概念](https://yeasy.gitbooks.io/docker_practice/swarm_mode/overview.html) 一节我们知道 `Swarm` 集群由 **管理节点** 和 **工作节点** 组成。本节我们来创建一个包含一个管理节点和两个工作节点的最小 `Swarm` 集群。

### 初始化集群

在 [`Docker Machine`](https://yeasy.gitbooks.io/docker_practice/machine) 一节中我们了解到 `Docker Machine` 可以在数秒内创建一个虚拟的 Docker 主机，下面我们使用它来创建三个 Docker 主机，并加入到集群中。

我们首先创建一个 Docker 主机作为管理节点。

```bash
$ docker-machine create -d virtualbox manager
```

我们使用 `docker swarm init` 在管理节点初始化一个 `Swarm` 集群。

```bash
$ docker-machine ssh manager

docker@manager:~$ docker swarm init --advertise-addr 192.168.99.100
Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

如果你的 Docker 主机有多个网卡，拥有多个 IP，必须使用 `--advertise-addr` 指定 IP。

>   执行 `docker swarm init` 命令的节点自动成为管理节点。

### 增加工作节点

上一步我们初始化了一个 `Swarm` 集群，拥有了一个管理节点，下面我们继续创建两个 Docker 主机作为工作节点，并加入到集群中。

```bash
$ docker-machine create -d virtualbox worker1

$ docker-machine ssh worker1

docker@worker1:~$ docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

This node joined a swarm as a worker.
$ docker-machine create -d virtualbox worker2

$ docker-machine ssh worker2

docker@worker1:~$ docker swarm join \
    --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
    192.168.99.100:2377

This node joined a swarm as a worker.
```

>   注意：一些细心的读者可能通过 `docker-machine create --help` 查看到 `--swarm*` 等一系列参数。该参数是用于旧的 `Docker Swarm`,与本章所讲的 `Swarm mode` 没有关系。

### 查看集群

经过上边的两步，我们已经拥有了一个最小的 `Swarm` 集群，包含一个管理节点和两个工作节点。

在管理节点使用 `docker node ls` 查看集群。

```bash
$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
03g1y59jwfg7cf99w4lt0f662    worker2   Ready   Active
9j68exjopxe7wfl6yuxml7a7j    worker1   Ready   Active
dxn1zf6l61qsb1josjja83ngz *  manager   Ready   Active        Leader
```



## 服务

我们使用 `docker service` 命令来管理 `Swarm` 集群中的服务，该命令只能在管理节点运行。

### 新建服务

现在我们在上一节创建的 `Swarm` 集群中运行一个名为 `nginx` 服务。

```bash
$ docker service create --replicas 3 -p 80:80 --name nginx nginx:1.13.7-alpine
```

注意，这里的replicas是服务（task）数，不一定与节点数相同，docker会自动进行负载均衡，比如再两个节点上创建十个服务，则docker会自动让他们负载均衡。单数时随机挑选一个。服务数可伸缩，见下方**服务伸缩**

现在我们使用浏览器，输入任意节点 IP ,即可看到 nginx 默认页面。

### 查看服务

使用 `docker service ls` 来查看当前 `Swarm` 集群运行的服务。

```bash
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                 PORTS
kc57xffvhul5        nginx               replicated          3/3                 nginx:1.13.7-alpine   *:80->80/tcp
```

使用 `docker service ps` 来查看某个服务的详情。

```bash
$ docker service ps nginx
ID                  NAME                IMAGE                 NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
pjfzd39buzlt        nginx.1             nginx:1.13.7-alpine   swarm2              Running             Running about a minute ago
hy9eeivdxlaa        nginx.2             nginx:1.13.7-alpine   swarm1              Running             Running about a minute ago
36wmpiv7gmfo        nginx.3             nginx:1.13.7-alpine   swarm3              Running             Running about a minute ago
```

使用 `docker service logs` 来查看某个服务的日志。

```bash
$ docker service logs nginx
nginx.3.36wmpiv7gmfo@swarm3    | 10.255.0.4 - - [25/Nov/2017:02:10:30 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.3.36wmpiv7gmfo@swarm3    | 10.255.0.4 - - [25/Nov/2017:02:10:30 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.3.36wmpiv7gmfo@swarm3    | 2017/11/25 02:10:30 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.255.0.4, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.99.102"
nginx.1.pjfzd39buzlt@swarm2    | 10.255.0.2 - - [25/Nov/2017:02:10:26 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.1.pjfzd39buzlt@swarm2    | 10.255.0.2 - - [25/Nov/2017:02:10:27 +0000] "GET /favicon.ico HTTP/1.1" 404 169 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:58.0) Gecko/20100101 Firefox/58.0" "-"
nginx.1.pjfzd39buzlt@swarm2    | 2017/11/25 02:10:27 [error] 5#5: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 10.255.0.2, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "192.168.99.101"
```

### 服务伸缩

我们可以使用 `docker service scale` 对一个服务运行的容器数量进行伸缩。

当业务处于高峰期时，我们需要扩展服务运行的容器数量。

```bash
$ docker service scale nginx=5
```

当业务平稳时，我们需要减少服务运行的容器数量。

```bash
$ docker service scale nginx=2
```

### 删除服务

使用 `docker service rm` 来从 `Swarm` 集群移除某个服务。

```bash
$ docker service rm nginx
```



更多<https://yeasy.gitbooks.io/docker_practice/swarm_mode/stack.html>