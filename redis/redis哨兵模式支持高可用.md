# 哨兵模式
哨兵是redis集群架构中非常重要的一个组件，主要功能如下
（1）集群监控，负责监控redis master和slave进程是否正常工作
（2）消息通知，如果某个redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员
（3）故障转移，如果master node挂掉了，会自动转移到slave node上
（4）配置中心，如果故障转移发生了，通知client客户端新的master地址

# quorum参数
1. 至少多少个哨兵同意，才能进行故障转移。
2. 故障转移的条件是，哨兵检测到master节点挂掉了。
3. 存活的哨兵节点需要进行一个分布式选举，选举出一个新的master节点。

# 两种数据丢失情况
**异步复制导致数据丢失**


**脑裂导致数据丢失** 

# sdown（主观宕机）和odown（客观宕机）转换机制  

# 三节点哨兵模式部署
## 哨兵的配置文件 sentinel.conf

每一个哨兵都可以去监控多个maser-slaves的主从架构
因为可能你的公司里，为不同的项目，部署了多个master-slaves的redis主从集群
相同的一套哨兵集群，就可以去监控不同的多个redis主从集群
你自己给每个redis主从集群分配一个逻辑的名称

sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5

sentinel monitor mymaster 127.0.0.1 6379 
类似这种配置，来指定对一个master的监控，给监控的master指定的一个名称，因为后面分布式集群架构里会讲解，可以配置多个master做数据拆分

sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
上面的三个配置，都是针对某个监控的master配置的，给其指定上面分配的名称即可

上面这段配置，就监控了两个master node

这是最小的哨兵配置，如果发生了master-slave故障转移，或者新的哨兵进程加入哨兵集群，那么哨兵会自动更新自己的配置文件

sentinel monitor master-group-name hostname port quorum

quorum的解释如下：

（1）至少多少个哨兵要一致同意，master进程挂掉了，或者slave进程挂掉了，或者要启动一个故障转移操作
（2）quorum是用来识别故障的，真正执行故障转移的时候，还是要在哨兵集群执行选举，选举一个哨兵进程出来执行故障转移操作
（3）假设有5个哨兵，quorum设置了2，那么如果5个哨兵中的2个都认为master挂掉了; 2个哨兵中的一个就会做一个选举，选举一个哨兵出来，执行故障转移; 如果5个哨兵中有3个哨兵都是运行的，那么故障转移就会被允许执行

down-after-milliseconds，超过多少毫秒跟一个redis实例断了连接，哨兵就可能认为这个redis实例挂了
parallel-syncs，新的master别切换之后，同时有多少个slave被切换到去连接新master，重新做同步，数字越低，花费的时间越多

假设你的redis是1个master，4个slave

然后master宕机了，4个slave中有1个切换成了master，剩下3个slave就要挂到新的master上面去

这个时候，如果parallel-syncs是1，那么3个slave，一个一个地挂接到新的master上面去，1个挂接完，而且从新的master sync完数据之后，再挂接下一个

如果parallel-syncs是3，那么一次性就会把所有slave挂接到新的master上去

failover-timeout，执行故障转移的timeout超时时长

## 安装

只要安装redis就可以了，不需要去部署redis实例的启动

wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz
tar -xzvf tcl8.6.1-src.tar.gz
cd  /usr/local/tcl8.6.1/unix/
./configure  
make && make install

使用redis-3.2.8.tar.gz（截止2017年4月的最新稳定版）
tar -zxvf redis-3.2.8.tar.gz
cd redis-3.2.8
make && make test
make install

2、正式的配置

哨兵默认用26379端口，默认不能跟其他机器在指定端口连通，只能在本地访问

mkdir /etc/sentinel
mkdir -p /var/sentinel/5000

/etc/sentinel/5000.conf

port 5000
bind 192.168.31.187
dir /var/sentinel/5000
sentinel monitor mymaster 192.168.31.187 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

port 5000
bind 192.168.31.19
dir /var/sentinal/5000
sentinel monitor mymaster 192.168.31.187 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

port 5000
bind 192.168.31.227
dir /var/sentinel/5000
sentinel monitor mymaster 192.168.31.187 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

3、启动哨兵进程

在eshop-cache01、eshop-cache02、eshop-cache03三台机器上，分别启动三个哨兵进程，组成一个集群，观察一下日志的输出
```
redis-sentinel /etc/sentinel/5000.conf
redis-server /etc/sentinel/5000.conf --sentinel
```
日志里会显示出来，每个哨兵都能去监控到对应的redis master，并能够自动发现对应的slave

哨兵之间，互相会自动进行发现，用的就是之前说的pub/sub，消息发布和订阅channel消息系统和机制

4、检查哨兵状态

redis-cli -h 192.168.31.187 -p 5000

sentinel master mymaster
SENTINEL slaves mymaster
SENTINEL sentinels mymaster

SENTINEL get-master-addr-by-name mymaster
