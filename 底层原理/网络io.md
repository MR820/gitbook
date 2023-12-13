### 面向TCP

基本命令

```shell
lsof  -op  1225 # 进程 PID 1225 打开的文件
netstat -natp # 内核级的网络连接状态
tcpdump -nn -i eth0 port 9090 # 抓包命令
```

服务端不需要为client的连接分配一个随机端口号

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_7b47d1c91a757c91d04a5f78bf1503cb.jpg)






![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_53ba62ba9c01654f51aa932c64c69445.jpg)

### 网络IO的变化模型

基本命令

```shell
strace -ff -o out #追踪系统调用
strace -ff -o out java TestSocket
```

多线程方式接受客户端连接 fd


#### BIO

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_b25a87997eab1e89cc2e9b17133395c1.jpg)

### 代码实例

[JAVA代码实例](https://github.com/MR820/java_io "JAVA代码实例")