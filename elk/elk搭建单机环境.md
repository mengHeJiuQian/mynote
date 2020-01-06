# 环境信息
1. 一台装centos7.7的机器
2. elk的三个软件统一使用7.5.1版本

# 创建elk用户
> 由于es不能以root用户身份启动，创建一个新用户elk用于启动es。

```bash
# 增加elk用户
$ useradd elk -m -U
# 说明：-m表示创建用户的home目录，-U 表示创建同名的组

# elk用户设置密码，如果密码小于8位，会提示“BAD PASSWORD: The password is shorter than 8 characters”
# 再次输入密码就会忽略掉该提示。
$ passwd elk

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

# 如果解压后es放置的目录需要root用户才能执行，需要使用chown命令将es目录的拥有者改为elk
# 因为es不能以root身份启动，修改掉es目录的拥有者后避免一些文件访问权限上的麻烦
$ sudo chmod -R elk elasticsearch-7.5.1/
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
进入es安装目录下的bin目录，用elk用户执行 ./elasticsearch启动

访问127.0.0.1:9200查看是否能访问并返回数据。


## 启动时遇到的错误
```txt
ERROR: [1] bootstrap checks failed
[1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[2]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

增大max_map_count操作如下：
修改/etc/sysctl.conf文件添加以下配置
vm.max_map_count = 655300
执行命令 sysctl -p 是修改的配置生效

增大max file descriptors操作如下：
查看当前每个进程最大同时打开文件数太小
$ ulimit -Hn # 硬限制
$ ulimit -Sn # 软限制
针对elk用户增加限制的数量：sudo vim /etc/security/limit.conf，文件末尾增加下面内容
elk soft nofile 65535
elk hard nofile 65535
```

# 安装logstash-7.5.1
解压、进入解压后logstash的目录，新建logs-path文件夹路径用于存储运行的日志。
进入logstash-7.5.1/config目录下，修改logstash.yml文件。修改后如下。
```
path.data: /usr/local/app/logstash-7.5.1/data
http.port: 9600
path.logs: /usr/local/app/logstash-7.5.1/logs-path
```

# 启动logstash
```
 bin/logstash -f config/logstash.conf
```






