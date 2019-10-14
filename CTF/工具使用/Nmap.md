# Nmap

参考连接：https://www.cnblogs.com/hanxiaobei/p/5603491.html

## 作用

- 检测网络中的存活主机
- 检测主机开放端口
- 检测主机系统、硬件地址、软甲版本
- 检测脆弱漏洞

## 扫描方式

- tcp反向的ident扫描
- ftp反弹扫描

### Tcp SYN Scan (sS)

这是一个基本的扫描方式,它被称为半开放扫描，因为这种技术使得Nmap不需要通过完整的握手，就能获得远程主机的信息。Nmap发送SYN包到远程主机，但是它不会产生任何会话.因此不会在目标主机上产生任何日志记录,因为没有形成会话。这个就是SYN扫描的优势.

如果Nmap命令中没有指出扫描类型,默认的就是Tcp SYN.但是它需要root/administrator权限.

### tcp的connection()扫描

如果不选择SYN扫描,TCP connect()扫描就是默认的扫描模式.不同于Tcp SYN扫描,Tcp connect()扫描需要完成三次握手,并且要求调用系统的connect().Tcp connect()扫描技术只适用于找出TCP和UDP端口.