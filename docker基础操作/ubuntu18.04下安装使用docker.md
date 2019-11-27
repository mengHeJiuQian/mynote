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

# 创建MySQL容器
docker run -p 33306:3306 -e MYSQL_ROOT_PASSWORD=1317598LY3201abc mysql

```
