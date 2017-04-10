
网络
------------------------------------

### 流程

    服务端
    socket->bind->listen->accept->read/print->shutdown->close

    客户端
    socket--------------->connect->read/print->shutdown->close

### socket

* `socket(SOCKET_FILEHANDLE, DOMAIN, TYPE, PROTOCOL)`
  * SOCKET_FILEHANDLE是句柄的名字，自定义
  * DOMAIN常见的有Unix域和Internet域
    * Unix域和AF_UNIX地址簇，如果是本地计算机的两个进程间进行通信，其地址簇（AF）就是AF_UNIX。在系统文件/usr/include/sys/socket.h中，将AF_UNIX地址簇赋值为常量1，也可以直接写裸字
    * Internet域和AF_UNIX地址簇，用于在不同计算机上的进程之间通信，AF_INET地址簇定义为常量2
  * TYPE指的是套接字的类型，常见的有两种SOCK_STREAM, SOCK_DGRAM，分别是流套接字和数据报套接字，即tcp和udp，同样可以直接写裸字
  * PROTOCOL是协议号的类型，可以将协议号赋值为0，以便允许系统为该套接字选择正确的协议

### bind

* `bind(SOCKET_FILEHANDLE, HAME)`
```perl
use Socket;
bind(COMM_SOCKET, socketaddr_un("/home/jody/eleie/perl/tserv"));
# 使用UNIX域时，套接字文件句柄COMM_SOCKET将和UNIX路径名绑定
bind(COMM_SOCKET, $socket_address);
# 使用Internet域时，套接字文件句柄COMM_SOCKET将绑定到套接字类型所含地址上
```

### listen

* `listen(SOCKET_FILEHANDLE, QUEUE_SIZE)`

listen函数负责在套接字上等待accept，并规定拒绝进一步请求之前所允许的排队连接总数，即客户端请求数目若超过了队列容量，则返回错误信息

### accept

accept函数会等待用户请求的到达，服务端进程接受来自客户端的请求，它会取队列中等待的第一个请求，并创建新的套接字文件句柄NEWSOCK，其用到的属性和RENDEZ_SOCK文件句柄相同，后者又称作一般套接字或集结套接字，即一开始服务端创建的套接字

### connect

connect函数用到了套接字文件句柄和要连接到的服务端地址
```perl
use Socket;
connect(CLIENT_SOCKET, socket_un("/home/fio/sock"));    # 使用UNIX域
connect(CLIENT_SOCKET, $packed_address);
# Ps. $packed_address相当于下面两种
$packed_address = pack('Sna4x8', $AF_INET, $port, $address);
$packed_address = sockaddr_in($port, inet_aton($ip_address);
```
### shutdown

shutdown函数可以按规定关闭套接字连接。或者也可以直接使用close

* `shutdown(SOCKET, HOW)`
  * HOW=0, 则在套接字上进一步的receive请求将遭拒绝
  * HOW=1，则在套接字上进一步的sends请求将遭拒绝
  * HOW=2，则所有的sends和receives都将停止

