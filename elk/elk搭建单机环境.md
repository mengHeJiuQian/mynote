# 环境信息
1. 一台装centos7.7的机器
2. elk的三个软件统一使用7.5.1版本

# 创建elk用户
> 由于es不能以root用户身份启动，创建一个新用户elk用于启动es。

```

useradd elk -m -U -p elk
```

