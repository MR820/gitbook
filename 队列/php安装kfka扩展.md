以kafka为例
### 一、下载扩展包
[pecl](https://pecl.php.net/package/rdkafka "pecl 上搜索kafka

![](https://oss.wyxxt.org.cn/images/2021/09/18/2bbc19c6ce73a2d6b9f6b0a56f4d82d2.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/81621446ad22704d18d0f0499a6a0700.png)

```shell
wget https://pecl.php.net/get/rdkafka-4.0.3.tgz
tar -zxf rdkafka-4.0.3.tgz
cd rdkafka-4.0.3
```

![](https://oss.wyxxt.org.cn/images/2021/09/18/d6f9e1327fded6ef1cefa3dedf6898cb.png)


**注意事项**
based on librdkafka
![](https://oss.wyxxt.org.cn/images/2021/09/18/1222ba28a17f3274162971a245bd4a46.png)


### 二、生成配置，生成.so文件

```shell
sudo /usr/local/bin/phpize

sudo ./configure --with-php-config=/usr/local/Cellar/php/7.3.10/bin/php-config

sudo make && make install
```

![](https://oss.wyxxt.org.cn/images/2021/09/18/616dc0aa78dbfff6e11af5a7a3dbfe09.png)

![](https://oss.wyxxt.org.cn/images/2021/09/18/b729c66ec1b2a7f6fd5646adf06e7fc5.png)

### 三、配置php.ini
```
extension = rdkafka.so
```
重启apache
![](https://oss.wyxxt.org.cn/images/2021/09/18/ccbc2583f96e7c66fcfb4c3dfb6b557f.png)