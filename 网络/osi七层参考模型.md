### osi七层模型

osi七层模型图，计算机网络的基石。

![](https://oss.wyxxt.org.cn/images/2021/09/18/ec5f704954a84de26ffaf652c419eaee.png)

### osi七层模型与tcp/ip四层模型关系

![](https://oss.wyxxt.org.cn/images/2021/09/18/9359ab16f5842b6019d37f19639181bc.png)



### osi七层和tcp/ip四层的区别



![](https://oss.wyxxt.org.cn/images/2021/09/18/b34e4c32b495fd994fc93174fdb78655.png)





![](https://oss.wyxxt.org.cn/images/2021/09/18/33fe112a8fa209f4a4cfbb4045266696.png)

### 常见协议或接口在哪一层

#### HTTP协议

HTTP是一个应用层协议，建立在TCP协议之上。
HTTP请求就是用来发送一段文本。关于这段文本如何组织，第一行写什么，第二行写什么，哪里加一个空行，就是HTTP协议所要规范的内容。

![](https://oss.wyxxt.org.cn/images/2021/09/18/4de2a8428d73b4c8e95f8bf32856c757.png)

#### Socket接口

HTTP的客户端和服务器，是通过Socket进行连接的。
**Socket是对osi模型第四层-传输层中的TCP/IP协议的封装**。Socket本身不是协议，而是**一个调用接口（API）**。应用层不必了解TCP/IP协议细节，直接通过对Socket接口函数的调用完成数据在IP网络的传输。

Socket包含了网络通信必须的五种信息：**连接使用的协议，本地主机的IP地址，本地进程的协议端口，远地主机的IP地址，远地进程的协议端口**。
因为有了Socket，客户端和服务端完全不需要了解底层细节，直接通过调用Socket来实现就可以了。
这也就是为什么服务器端**可以获取到客户端的IP地址**的原因，因为Socket中包含了远地主机的IP地址。（当然，通过代理服务器进行访问的除外，这种要依靠HTTP协议的X-Forwarded-For头来确认IP，不在本次的讨论范围中）

那为什么**无法获取到客户端的MAC地址**呢？很简单，因为Socket中无法取到MAC地址。

### 当打开一个URL时，究竟发生了什么
当我们在浏览器中打开一个链接后，osi各层到底发生了什么？
这里撇开DNS解析之类的东西，只探讨HTTP报文的发送

1. 发送端
应用层数据其实只有那么几行文本，然后往下，每过一层，都要被加上首部/尾部。这个过程就像是一层一层的穿衣服。

2. 数据流转
数据一般要经过交换机、路由器等网络设备，层层转发，这些设备所做的事情就像是： 脱掉一件或几件衣服，做一些修修补补，然后再重新穿回去。

>以L2交换机为例看一下，因为L2交换机会认别到帧这一层，记录/学习MAC地址，并将帧发送到目的地。
>![](https://oss.wyxxt.org.cn/images/2021/09/18/2144a4610a96b268e31a3d289cb1969f.png)

>路由器。当一个LAN希望连接到另一个LAN时，就必须使用路由器设备了。当然，可以通过构建超大型的LAN，来避免使用路由器，但这时，交换机就需要管理大量的MAC地址，同时进行大量的广播通信，设备的负担就会相当大。路由器会进行路由选择，让数据达到下一跳。经过路由器后，为了能到达下一跳，数据链路层中的MAC地址就被篡改了
>![](https://oss.wyxxt.org.cn/images/2021/09/18/df860378778fb241694acd0f18cc74f2.png)

3. 接收端
WEB服务器所在的主机。将衣服一件一件全部脱掉 ，最后WEB服务器就取到了最初的应用层数据。