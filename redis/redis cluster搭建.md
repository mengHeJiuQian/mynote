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
bind 192.168.31.187		
appendonly yes
```

```
cp redis_6379 redis_7001
vi redis_7001
在redis_7001中修改下PORT参数为7001
```

创建redis cluster的脚本用ruby写的，需要先安装ruby。cluster的创建只需要在一台机器上执行就行了。所以一下命令在一台机器上执行即可。
```shell
yum install -y ruby
yum install -y rubygems
gem install redis

cp /usr/local/redis-3.2.8目录/src/redis-trib.rb /usr/local/bin
```