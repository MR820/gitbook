<h3>顺序写入</h3>
<p>减少寻址时间</p>
<h3>mmap</h3>
<p>减少io，写入性能逼近内存</p>
<img src="https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_804b05e693fea4f3247bfbde861be865.jpg" alt="" />
<h3>DMA（Direct Memory Access）</h3>
<p>解放cpu</p>
<img src="https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_66e53310154eb776f2f10895ab1dd5a2.jpg" alt="" />
<img src="https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_629167b34f5b7073fc5bc9ccc9f8070c.jpg" alt="" />
<h3>零拷贝</h3>
<p>节省将磁盘数据拷贝到用户空间的性能损失</p>
<h4>一般情况</h4>
<img src="https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_521275edb72fcd68478c9e5a33b89721.jpg" alt="" />
<h4>Zero拷贝</h4>
<img src="https://oss.wyxxt.org.cn/images/2021/09/18/wp_editor_md_1a5a0d9975b74a963dc234d759c4bdd2.jpg" alt="" />