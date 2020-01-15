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
## 查看相关操作
```bash
# 查看kafka所有的topic
$ ./kafka-topics.sh --zookeeper localhost:2181 --list

# 启动一个消费者，从头消费所有消息，可以用来查看topic所有的消息
$ ./kafka-console-consumer.sh --bootstrap-server localhost:2181 --topic nginx_log --from-beginning

# 查看Topic的详细信息
$ kafka-topics.sh --zookeeper localhost:2181 --topic  __consumer_offsets  --describe
```

## 删除相关操作
```bash
# 删除一个Topic
$ kafka-topics.sh --zookeeper localhost:2181 --delete --topic TOPIC_NAME

```