
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

> 攻击机：Kali2016   ip：192.168.248.145
>
> 目标主机：windows 7  ip：192.168.248.148

实验步骤：
----------

把subterfuge复制到攻击机 

![1.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![1.png](http://image.3001.net/images/20170327/14905916847515.png!small)
使用解压 

    tar zxvf SubterfugePublicBeta5.0.tar.gz 

![2.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![2.png](http://image.3001.net/images/20170327/14905917968085.png!small)
进入压缩包查看文件

![3.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![3.png](http://image.3001.net/images/20170327/1490591881457.png!small)
查看说明文件

![4.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![4.png](http://image.3001.net/images/20170327/14905919143503.png!small)
提示使用 python install.py –i进行安装

![5.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![5.png](http://image.3001.net/images/20170327/14905919626803.png!small)
### 使用

python installer\_old.py –i  命令行安装

python install.py –i 图形界面安装（出现命名失败，是因为我安装过一次）

![7.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![7.png](http://image.3001.net/images/20170327/14905920801371.png!small)
安装完成后进去subterfuge默认安装目录 /usr/share/subterfuge使用

    ./sslstrip.py –h

### 进行测试

![8.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![8.png](http://image.3001.net/images/20170327/14905921439469.png!small)
开启网卡转发

    echo “1” >/proc/sys/net/ipv4/ip_forword

![9.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![9.png](http://image.3001.net/images/20170327/14905921797613.png!small)
使用iptables把80端口的数据转发到1234端口，方便sslstrip进行监听 

![10.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![10.png](http://image.3001.net/images/20170327/14905922115326.png!small)
### 启用sslstrip进行

-a 记录所有来这SSL和HTTP的数据

-l 1234 监听来自1234端口的数据与之前设置的iptables端口号一样

![11.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![11.png](http://image.3001.net/images/20170327/14905922446371.png!small)
已成功启动sslstrip进行监听，现在启动kali自带的tcpdump进行嗅探并捕获转发的报文

![12.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![12.png](http://image.3001.net/images/20170327/14905922722787.png!small)
目标机的ip地址

在攻击机上执行   

     tcpdump –A –i eth0tcp and src 192.168.248.148 and port 80

-A 开启ASCALL模式，把捕获的数据已ACSALL模式显示

-i 选择监听的网卡

在目标机上登陆https：//xxx.comtcp 抓取所有的tcp协议的数据包

src 192.168.248.148 and port 80 抓取源地址和端口号的数据

![13.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![13.png](http://image.3001.net/images/20170327/14905923261368.png!small)
![14.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![14.png](http://image.3001.net/images/20170327/1490592326668.png!small)
在目标机上登陆https：//xxx.com

![15.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![15.png](http://image.3001.net/images/20170327/1490592367400.png!small)
![16.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![16.png](http://image.3001.net/images/20170327/14905923673068.png!small)
能捕获163的用户名和Cookie等信息，而目标无任何安全提示

![17.png](http://www.freebuf.com/buf/themes/freebuf/images/grey.gif)

![17.png](http://image.3001.net/images/20170327/14905924229029.png!small)
> 墙外下载地址：<https://code.google.com/archive/p/subterfuge/downloads>
>
> 百度云链接：<http://pan.baidu.com/s/1sl6PWSH> 密码：t831

> 原文地址: <http://www.freebuf.com/sectool/130456.html>
