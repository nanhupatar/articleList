docker 安装 mongodb

定义数据卷

```sh
docker volume create mongo
```

下载镜像

```Dockerfile
docker run -p 27017:27017 -v mongo:/data/db --name mongodb -d mongo
```
