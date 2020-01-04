# 环境信息
1. 一台装centos7.7的机器
2. elk的三个软件统一使用7.5.1版本

# 创建elk用户
> 由于es不能以root用户身份启动，创建一个新用户elk用于启动es。

```bash
# 增加elk用户
$ useradd elk -m -U -p elk
# 说明：-m表示创建用户的home目录，-U 表示创建同名的组，-p表示密码

# 将用户elk加入到/etc/sudoers文件中，让elk用户具有和root一样高的权限
$ whereis sudoers
sudoers: /etc/sudoers /etc/sudoers.d /usr/share/man/man5/sudoers.5.gz

$ ls -l /etc/sudoers # 发现没有写权限，增加当前用户对该文件的写权限
-r--r----- 1 root root 4328 Oct 24 23:23 /etc/sudoers
$ chmod u+w /etc/sudoers

$ vim /etc/sudoers
## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
elk     ALL=(ALL)       ALL # 增加这一行

# 接下来使用elk用户进行安装es的操作
$ su - elk 
```

# 安装elasticsearch-7.5.1
```bash
# 解压
$ tar -axvf elasticsearch-7.5.1-linux-x86_64.tar.gz 
```
进入解压后的目录下的config目录，修改elasticsearch.yml文件，下面是修改后的yml文件内容
```txt
# ---------------------------------- Cluster -----------------------------------
cluster.name: es

# ------------------------------------ Node ------------------------------------
node.name: node-1

# ----------------------------------- Paths ------------------------------------
path.data: /usr/local/app/elasticsearch-7.5.1/data
path.logs: /usr/local/app/elasticsearch-7.5.1/logs

# ----------------------------------- Memory -----------------------------------
# 启动时锁定内存
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

# ---------------------------------- Network -----------------------------------
network.host: 0.0.0.0
http.port: 9200

# --------------------------------- Discovery ----------------------------------
cluster.initial_master_nodes: ["node-1"]
```

# 启动es
进入es安装目录下的bin目录，用elk用户执行 ./elasticsearch
```bash
$ su - elk
./

```


