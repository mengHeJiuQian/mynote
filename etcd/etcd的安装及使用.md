# ubuntu18安装etcd

```
获取etcd软件包
$ wget https://github.com/etcd-io/etcd/releases/download/v3.3.18/etcd-v3.3.18-linux-amd64.tar.gz

解压
$ tar -zxvf etcd-v3.3.18-linux-amd64.tar.gz

进到解压出来的新目录，将目录下的etcd和etcdctl二进制文件移动到/usr/local/bin目录，
/usr/local/bin要在PATH环境变量中，如果没有请自行配置，自行配置如下。
在/etc/profile文件中增加 export PATH=${PATH}:/usr/local/bin



```