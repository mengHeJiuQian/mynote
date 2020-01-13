# 启动/关闭 kafka
```bash
# 启动kafka，不使用kafka自带的zookeeper
# -daemon是后台启动
$ bin/kafka-server-start.sh -daemon config/server.properties

# 如果使用kafka自带的zookeeper，需要先后台启动zookeeper
$ ./zookeeper-server-start.sh -daemon ../config/zookeeper.properties

# 关闭kafka
$ bin/kafka-server-stop.sh 
```

# kafka的操作topic
```bash
# 查看kafka所有的topic
$ ./kafka-topics.sh --zookeeper localhost:2181 --list

# 查看topic内容
$ ./kafka-console-consumer.sh --zookeeper localhost:2181 --topic nginx_log --from-beginning
```