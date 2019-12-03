# 安装过程记录
```txt
集群本地root用户密码是root
虚拟机用户sheldon密码是sheldon

最小化安装centos7虚拟机后是不能上网的
编辑网卡文件
vi /etc/sysconfig/network-scripts/ifcfg-ens0s3
修改onboot属性为onboot=yes
保存文件后,重启 网卡 service network restart

再次编辑/etc/sysconfig/network-scripts/ifcfg-ens0s3
增加如下属性


```