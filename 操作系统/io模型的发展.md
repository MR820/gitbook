### 计算机组成原理

kernel统一管理io


app访问io通过调用kernel（不是直接调用），kernel有保护。cpu执行app碰到io操作（有中断命令），系统调用   保护现场（将数据压入app内存空间），恢复现场将数据从app内存空间读取出来。**系统调用涉及用户态内核态切换，消耗性能**




![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_16214b8491d8d98668ab9f5d2cdf2ddd.jpg)






### io演化模型








![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_970a6550a24d036e79a53c3792c21292.jpg)






#### bio

单线程
accept() 阻塞1
read() 阻塞2
**优化**
每连接每线程
accept到连接后抛出一个线程负责read，阻塞互不影响





![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_301c7adf3835016cdff57573c5c9f889.jpg)






#### nio







![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_17e2f85f421d9b8e78e335175593891c.jpg)





#### select poll




![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_5b3a6a9b49eaeced12b02585e2e27e4f.jpg)







#### epoll





![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_a3a31defa63c507b5d67bee1f8b25c03.jpg)