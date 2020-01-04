# 环境信息
1. 一台装centos7.7的机器
2. elk的三个软件统一使用7.5.1版本

# 创建elk用户
> 由于es不能以root用户身份启动，创建一个新用户elk用于启动es。

```bash
# 增加elk用户
$ useradd elk -m -U -p elk
# 说明：-m表示创建用户的home目录，-U 表示创建同名的组，-p表示密码

# 将用户elk加入到/etc/sudoers文件中，让elk用户具有和root一样高的权限
$ whereis sudoers
sudoers: /etc/sudoers /etc/sudoers.d /usr/share/man/man5/sudoers.5.gz

$ ls -l /etc/sudoers # 发现没有写权限，
-r--r----- 1 root root 4328 Oct 24 23:23 /etc/sudoers
$ chmod u+w /etc/sudoers

$ vim /etc/sudoers

```

