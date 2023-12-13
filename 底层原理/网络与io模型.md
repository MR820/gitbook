# 常见问题

[tcp-ip协议](https://wyxxt.org.cn/archives/网络通信-tcp-ip模型详解.html)


![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_2c56594eaeaf453af80cb0256bac0208.jpg)



## TCP三次握手

[三次握手](https://wyxxt.org.cn/archives/网络通信-tcp-ip模型详解.html#3)

/proc/sys/net/core/somaxconn ACCEPT队列  与程序接口设置的backlog 取min

每个人顾好自己，每个人处理好对别人的打扰

/proc/sys/net/ipv4/tcp_max_syn_backlog DDOS

backlog满了（accept队列满了），新客户端直接Connection refused

## TCP四次分手

[四次分手](https://wyxxt.org.cn/archives/网络通信-tcp-ip模型详解.html#4)

## TCP连接状态

[TIME_WAIT,FIN_WAIT2,CLOSE_WAIT](https://wyxxt.org.cn/archives/网络io-2.html)

## Connection refused

## OSI七层参考模型

[osi七层参考模型](https://wyxxt.org.cn/archives/osi七层模型与tcp-ip协议.html)

## 什么是长连接和短连接

TCP是长连接吗？

tcp只是连接，受应用层协议

连接是不是一个复用载体

举一个例子：http1.0,1.1没有开启keepalive保持，连接只负责一次同步阻塞的请求+响应,短连接

举一个例子：Http1.0，1.1开启了keepalive保持，同步复用连接：多次（请求+响应）

举一个例子：dubbo协议（rpc），打开连接，同步/异步复用连接（请求+响应）（请求请求+响应响应），当复用连接的时候，异步复用消息的ID，而且服务端和客户端同时完成这个约束 有状态通信

## IO模型

0: IO是程序对内核的socket-queue的包装

BIO：读取，一直等queue里有才返回，阻塞模型，每连接对用每线程

NIO：读取，立刻返回：两种结果，读到，没读到，程序逻辑要自己维护，NIO noblock

多路复用器：内核增加select，poll，epoll新增的和数据接收，连接接收实质无关的方法调用，得到对用socket的事件（listen-socket，eastablished-socket），可以优先地再次accept，R/W

AIO*

## 同步阻塞，同步非阻塞

BIO，NIO，多路复用器，在IO模型上都是同步的，都是程序自己accpet，R/W

## 有状态，无状态

举一个例子：dubbo协议（rpc），打开连接，同步/异步复用连接（请求+响应）（请求请求+响应响应），当复用连接的时候，异步复用消息的ID，而且服务端和客户端同时完成这个约束 有状态通信

## 粘包，拆包

有程序，有内核，程序和内核协调工作

有些是内核做的事情，三次握手，数据发送出去，接收进来，内核，TCP，分包

到我们自己的程序，即便在一个socket里，也可能收到多个消息在一个字节数组中， 我们要自己拆解