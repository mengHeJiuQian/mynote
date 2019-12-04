# 持久化
**AOF**
> AOF（Append Only File）持久化方式是把redis执行过的所有命令保存到AOF文件中，恢复数据时把AOF文件中的命令重新执行一遍即可恢复redis中的绝大部分数据。丢失的部分内存数据是因为执行的redis命令追加到文件的过程会有延迟。

**AOF持久化大致过程**
1. redis将修改

