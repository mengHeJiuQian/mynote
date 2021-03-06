# 安装Centos7过程记录
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
yum makecache #生成最新的元数据缓存

（4）安装一些工具
yum install -y wget net-tools lrzsz

（5）安装JDK
解压tar.gz，配置环境变量vi /etc/profile
export JAVA_HOME=/usr/local/app/jdk8.xxxxxx
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

让环境变量生效
source /etc/profile

验证Java是否安装配置完善 
java -version
会打印出Java的一些信息

（6）安装Perl
安装gcc之类的工具，编译安装Perl的过程会使用gcc
yum install -y gcc

wget https://www.cpan.org/src/5.0/perl-5.28.1.tar.gz
tar -xzf perl-5.28.1.tar.gz
cd perl-5.28.1
./Configure -des -Dprefix=/usr/local/perl
make
make test
make install
perl -v

（7）在4个虚拟机中安装CentOS集群

（1）按照上述步骤，再安装三台一模一样环境的linux机器
（2）另外三台机器的hostname分别设置为eshop-cache02，eshop-cache03，eshop-cache04
（3）安装好之后，在每台机器的hosts文件里面，配置好所有的机器的ip地址到hostname的映射关系

有一个简单的不用重复安装虚拟机的方式，将安装好的虚拟机关闭，右键“复制”，选择虚拟机存放路径，选择全部重新生成网卡，下一步，选择完全复制。就OK了。
```

# 配置4台CentOS为ssh免密码互相通信
```txt
（1）主机映射配置
在/etc/hosts文件里添加IP和主机名的映射
192.168.16.171 eshop-cache01
192.168.16.172 eshop-cache02
192.168.16.173 eshop-cache03
192.168.16.174 eshop-cache04

在一台机器上配置好映射后，复制本地的/etc/hosts文件到其他机器上的/etc目录下
[root@eshop-cache01 ~]# scp /etc/hosts root@192.168.16.174:/etc/

（2）配置机器之间免密码通信
生成ssh秘钥，生成的信息在~/.ssh/下
ssh-keygen -t rsa
将对外的秘钥写到~/.ssh/authorized_keys文件里，没authorized_keys文件就创建。
cat id_rsa.pub  > authorized_keys

在另外三台的机器节点上也生成各自的authorized_keys，并使用ssh-copy-id -i hostname命令将各自机器里生成的公钥拷贝到指定第一台机器的authorized_keys文件中
eshop-cache02里执行ssh-copy-id -i eshop-cache01
eshop-cache03里执行ssh-copy-id -i eshop-cache01
eshop-cache04里执行ssh-copy-id -i eshop-cache01

此时eshop-cache01主机的authorized_keys就有了四个主机的公钥，将eshop-cache01的authorized_keys拷贝到另外三台机器的~/.ssh/目录下。以上，四台主机之间实现了相互之间免密登录。
scp /root/.ssh/authorized_keys root@eshop-cache02:/root/.ssh/
scp /root/.ssh/authorized_keys root@eshop-cache03:/root/.ssh/
scp /root/.ssh/authorized_keys root@eshop-cache04:/root/.ssh/

验证免密登录效果
ssh 主机名，可以登录并且不需要输入密码，完成。



```