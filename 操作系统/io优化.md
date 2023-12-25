### 顺序写入

减少寻址时间

### mmap

减少io，写入性能逼近内存

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_804b05e693fea4f3247bfbde861be865.jpg)

### DMA（Direct Memory Access)

解放cpu

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_66e53310154eb776f2f10895ab1dd5a2.jpg)![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_629167b34f5b7073fc5bc9ccc9f8070c.jpg)

### 零拷贝

节省将磁盘数据拷贝到用户空间的性能损失

#### 一般情况

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_521275edb72fcd68478c9e5a33b89721.jpg)

#### Zero拷贝

![](https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_1a5a0d9975b74a963dc234d759c4bdd2.jpg)