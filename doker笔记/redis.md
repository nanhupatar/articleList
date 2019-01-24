docker redis 教程

创建数据卷，持久化存储

```sh
docker volume creat redis
```

pull 镜像并运行容器

```sh
docker run -p 6379:6379 -d -v redis:/data --name redis redis
```
