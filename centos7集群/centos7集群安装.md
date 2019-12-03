# 安装过程记录
```txt
集群本地root用户密码是root
虚拟机用户sheldon密码是sheldon
（1）配置网络
最小化安装centos7虚拟机后是不能上网的
编辑网卡文件
vi /etc/sysconfig/network-scripts/ifcfg-ens0s3
修改onboot修改属性为
onboot=yes
BOOTPROTO=dhcp
保存文件后,重启 网卡 service network restart

再次编辑/etc/sysconfig/network-scripts/ifcfg-ens0s3
增加如下属性,已有的相同属性配置直接修改即可.这里的IP、网关根据自己的主机查看，执行ifconfig就可看到。
BOOTPROTO=static
IPADDR=192.168.16.X
NETMASK=255.255.255.0
GATEWAY=192.168.16.1
DNS1=114.114.114.114
DNS2=8.8.8.8

保存后重启网络 service network restart

（2）关闭防火墙
方便每台虚拟机相互通信，防止因为防火墙造成连接失败，关闭防火墙
service iptables stop
service ip6tables stop
chkconfig iptables off
chkconfig ip6tables off

vi /etc/selinux/config 
SELINUX=disabled

（3）配置yum
yum clean all
yum makecache




```