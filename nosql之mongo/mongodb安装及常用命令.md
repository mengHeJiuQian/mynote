[toc]

# 通过docker安装mongo4.0
```bash
# 拉取镜像
$ docker pull mongo:4.0-xenial

# 查看下载的tag为4.0-xenial的镜像
$ docker images

# 启动一个运行mongodb的容器
$ docker run -p 27017:27017  \
--name mongodb_4.0 \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=1317598LY3201abc \
-d \ 镜像id
# 使用 -p 将容器的27017端口映射到宿主机的27017端口；
# 使用 -v 将宿主机当前路径的db文件夹挂载至容器中，将数据持久化到宿主机。
# 使用-e 设置一些root用户名和密码的初始化值
# 使用-d 将后台模式启动。
# 进入容器
$ docker exec -it 容器ID /bin/bash

```

# mongo常用命令
```txt
show dbs

use test

# 在test创建一个用户
db.createUser({user:"sheldon",pwd:"sheldon",roles:[{role:"dbAdmin",db:"test"}]})

```