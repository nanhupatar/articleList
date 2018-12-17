> 一直想研究一下后端的知识，先从 docker 入手吧。本文是腾讯云实验室 docker 教程的笔记

### 安装 docker

Docker 软件包已经包括在默认的 CentOS-Extras 软件源里。因此想要安装 docker，只需要运行下面的 yum 命令：

```shell
yum install docker-io -y
```

成功后运行使用命令查看版本

```shell
docker -v
```

启动 docker

```shell
service docker start
```

设置开机自启动

```shell
chkconfig docker on
```

### 配置 docker

因为国内访问 Docker Hub 较慢, 可以使用腾讯云提供的国内镜像源, 加速访问 Docker Hub

依次执行以下命令

```shell
echo "OPTIONS='--registry-mirror=https://mirror.ccs.tencentyun.com'" >> /etc/sysconfig/docker
```

```shell
systemctl daemon-reload
```

```shell
service docker restart
```

### docker 简单操作

下载一个官方的镜像到本地

```shell
docker pull centos
```

下载好的镜像就会出现到镜像列表里

```shell
docker images
```

### 运行容器

以刚才生成的 centos 镜像为例进行操作。

生成一个 centos 镜像为模板的容器并使用 bash shell

```shell
docker run -it centos /bin/bash
```

这个时候可以看到命令行的前端已经变成了[root@(一串 hash id)]的形式，这说明我们已经成功进入了 centos 容器

在容器内执行任意命令, 不会影响到宿主机, 如下

```shell
mkdir -p /data/simple_docker
```

可以看到 /data 目录下已经创建成功了 simple_docker 文件夹

```shell
ls /data
```

退出容器

```shell
exit
```

查看宿主机的 /data 目录, 并没有 simple_docker 文件夹, 说明容器内的操作不会影响到宿主机

```shell
ls /data
```

### 保存容器

查看所有的容器信息，能获取容器的 id

```shell
docker ps -a
```

然后执行如下命令，保存镜像

```shell
docker commit -m="第一个docker镜像" do44ea77783 centos
```
