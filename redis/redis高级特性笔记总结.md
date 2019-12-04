# 持久化
**AOF**
> AOF（Append Only File）持久化方式会把redis执行过的所有命令保存到AOF文件中，恢复数据时把AOF文件中的命令重新执行一遍即可恢复到redis上一次