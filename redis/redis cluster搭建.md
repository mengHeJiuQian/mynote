#
```
mkdir -p /etc/redis-cluster
mkdir -p /var/log/redis
mkdir -p /var/redis/7001
```

修改redis.conf文件
```
port 7001
cluster-enabled yes
cluster-config-file /etc/redis-cluster/node-7001.conf
cluster-node-timeout 15000
daemonize	yes	
pidfile		/var/run/redis_7001.pid 						
dir 		/var/redis/7001		
logfile /var/log/redis/7001.log
bind 192.168.199.171		
appendonly yes
```

```
cp redis_6379 redis_7001
vi redis_7001
在redis_7001中修改下PORT参数为7001
```

redis5以前的版本集群是依靠ruby脚本redis-trib.rb实现，需要先安装ruby。cluster的创建只需要在一台机器上执行就行了。所以一下命令在一台机器上执行即可。
**redis5之前版本创建方式**
```shell
yum install -y ruby
yum install -y rubygems
gem install redis

cp /usr/local/redis-3.2.8目录/src/redis-trib.rb /usr/local/bin

redis-trib.rb create --replicas 1 192.168.16.171:7001 192.168.16.171:7002 192.168.16.172:7003 192.168.16.172:7004 192.168.16.173:7005 192.168.16.173:7006

--replicas: 每个master有几个slave

6台机器，3个master，3个slave，尽量自己让master和slave不在一台机器上

yes

redis-trib.rb check 192.168.16.171:7001

```

**redis5及之后版本创建方式**
```shell
$ redis-cli --cluster create --cluster-replicas 1 192.168.199.171:7001 192.168.199.171:7002 192.168.199.172:7003 192.168.199.172:7004 192.168.199.173:7005 192.168.199.173:7006
创建期间出现的选项点击yes
```

# 系统及参数调整
```
redis cluster启动时有如下日志：增加应用打开文件的最大个数为10032。
Increased maximum number of open files to 10032 (it was originally set to 1024).

ulimit -a 可以显示一些系统限制。
执行 ulimit -n 10032，设置应用打开文件的最大个数为10032。
```

# 验证redis cluster
```bash
# -a访问服务端密码，-c表示集群模式，指定ip地址和端口号
$ redis-cli -c -a xxx -h 192.168.199.171 -p 7001

# 进入之后执行如下
>  cluster info（查看集群信息）
> cluster nodes（查看节点列表）
```


4、读写分离+高可用+多master

读写分离：每个master都有一个slave
高可用：master宕机，slave自动被切换过去
多master：横向扩容支持更大数据量