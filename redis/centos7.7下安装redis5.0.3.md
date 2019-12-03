# 安装tcl
```
redis安装之前需要先安装tcl工具
wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz
tar -xzvf tcl8.6.1-src.tar.gz
cd  /.../tcl8.6.1/unix/
./configure  
make && make install
```

#安装redis
```txt
redis下载网址 https://redis.io/
解压redis软件包
tar -zxvf redis-5.0.3.tar.gz 
cd redis-5.0.3
make && make test && make install
```