[toc]

# docker的安装过程
```bash
# 使用yum安装，用于前期学习足矣。
yum upgrade
yum install docker

# 获取最新版本的mysql镜像
docker pull mysql

# 列出本地所有镜像
docker images

# 创建一个MySQL容器
docker run -p 33306:3306 -e MYSQL_ROOT_PASSWORD=1317598LY3201abc mysql

# 进入MySQL容器,登陆MySQL
docker exec -it 容器id(CONTAINER ID) /bin/bash
# 登录之后修改root用户可以在任意机器上连接MySQL
mysql> GRANT ALL ON *.* TO 'root'@'%';
mysql> flush privileges;

# 停止一个正在运行的docker容器
docker stop CONTAINER_ID
# 给docker容器重新命名
docker rename old_name new_name
```
