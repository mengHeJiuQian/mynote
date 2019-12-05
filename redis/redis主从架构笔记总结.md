# 主从架构
> 一个redis集群的master节点专门处理写请求，将数据同步到其他的slave节点，其他的节点只处理读请求，支持水平扩展的读高并发架构可以达到提升redis读性能的目的。

当启动一个slave node的时候，它会发送一个**PSYNC**命令给master node。如果这是slave node重新连接master node，那么master node仅仅会复制给slave部分缺少的数据; 否则如果是slave node第一次连接master node，那么会触发一次**full resynchronization**。

开始full resynchronization的时候，master会启动一个后台线程，开始生成一份RDB快照文件，同时还会将从客户端收到的所有写命令缓存在内存中。RDB文件生成完毕之后，master会将这个RDB发送给slave，slave会先写入本地磁盘，然后再从本地磁盘加载到内存中。然后master会将内存中缓存的写命令发送给slave，slave也会同步这些数据。

slave node如果跟master node有网络故障，断开了连接，会自动重连。master如果发现有多个slave node都来重新连接，仅仅会启动一个rdb save操作，用一份数据服务所有slave node。

## 主从复制涉及到的三个技术点
**主从复制的断点续传**

从redis 2.8开始，就支持主从复制的断点续传，如果主从复制过程中，网络连接断掉了，那么可以接着上次复制的地方，继续复制下去，而不是从头开始复制一份。

master node会在内存中常见一个**backlog**，master和slave都会保存一个**replica offset**还有一个master id，offset就是保存在backlog中的。如果master和slave网络连接断掉了，slave会让master从上次的replica offset开始继续复制。

但是如果没有找到对应的offset，那么就会执行一次resynchronization

**无磁盘化复制**

master在内存中直接创建rdb，然后发送给slave，不会在自己本地落地磁盘了。这里涉及redis.conf文件两个配置。
```
repl-diskless-sync
repl-diskless-sync-delay    #等待一定时长再开始复制，因为要等更多slave重新连接过来
```

**过期key处理**

slave不会过期key，只会等待master过期key。如果master过期了一个key，或者通过LRU淘汰了一个key，那么会模拟一条del命令发送给slave。

# 主从架构环境搭建
```
redis安装之前需要先安装tcl工具
wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz
tar -xzvf tcl8.6.1-src.tar.gz
cd  /.../tcl8.6.1/unix/
./configure  
make && make install

redis下载网址 https://redis.io/
解压redis软件包
tar -zxvf redis-5.0.3.tar.gz 
cd redis-5.0.3
make 
make test 
make install PREFIX=/usr/local/app/redis-bin
redis安装完成，接下来是配置redis开机自启
把redis作为一个系统的daemon进程去运行的，每次系统启动，redis进程一起启动
（1）在解压的redis目录下的utils目录下，有个redis_init_script脚本文件
（2）将redis_init_script脚本拷贝到/etc/init.d目录下，将redis_init_script重命名为redis_6379，6379是我们希望这个redis实例监听的端口号
（3）修改redis_6379脚本中的REDISPORT配置参数，设置为相同的端口号（默认就是6379）
（4）创建两个目录：/etc/redis（存放redis的配置文件），/var/redis/6379（存放redis的持久化文件）
（5）修改redis配置文件（默认在redis解压目录下，redis.conf），拷贝到/etc/redis目录中，修改名称为6379.conf
修改6379.conf中的部分配置为生产环境
daemonize	yes							让redis以daemon进程运行
pidfile		/var/run/redis_6379.pid 	设置redis的pid文件位置
port		6379						设置redis的监听端口号
dir 		/var/redis/6379				设置持久化文件的存储位置
（6）启动redis，执行cd /etc/init.d, chmod 777 redis_6379，./redis_6379 start
（7）确认redis进程是否启动，ps -ef | grep redis
（8）让redis跟随系统启动自动启动
在redis_6379脚本中，最上面，加入两行注释
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
chkconfig redis_6379 on

以上配置和master节点配置没有什么不同，slave需要配置主节点和强制只读操作。
Redis5.0所以配置不是slaveof而是replicaof
replicaof eshop-cache01 6379
replica-read-only yes 从节点

```
