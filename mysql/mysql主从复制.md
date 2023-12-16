### 选择方案
1. 基于GTID复制(Global Transaction ID)
2. MHA架构

### 一、GTID复制实现
#### 1、在主DB服务器上建立复制账号
```shell
create user 'repl' @ 'IP段' identified by 'PassW0rd'; # 创建账号
GRANT REPLICATION SLAVE ON *.* TO 'repl' @ 'IP段'; # 授权

# mysql 8.0之前
grant all privileges on db_name.* to db_user@'%' identified by 'db_pass'; #授权语句，特别注意有分号
```

#### 2、开启binlog日志,并配置binlog_format=row,binlog_row_image=minimal
```shell
log-bin=/var/log/mysql-bin
server-id=1

/etc/init.d/mysql restart

mysql -u root -p
set session binlog_format=row;
set session binlog_row_image=minimal;
```

### 3、配置主服务器
```shell
vim /etc/mysql/mysqld.conf.d/mysqld.cnf
```
![](https://oss.wyxxt.org.cn/images/2021/09/18/6e3999db0bfa4dda4dc23f311f27bc4b.png)
```shell
/etc/init.d/mysql restart
```

### 4、配置从服务器
![](https://oss.wyxxt.org.cn/images/2021/09/18/9d9ff3766ed220ed988bb42fe3fa804d.png)

### 5、主服务器数据同步到从服务器
```shell
# 在主服务器上
mysqldump --single-transaction --master-data=2 --triggers --routines --all-databases -uroot -p > all.sql
scp ./all.sql slave1@192.168.1.120:/home/slave1/
# 在从服务器上
mysql -uroot -p < all.sql
```

```shell
mysql -uroot -p

change master to master_host='192.168.1.103',master_user='repl',master_password='123456',master_auto_position=1;

start slave;
show slave status \G;

```
**发现连接数据库失败，修改配置bind_ip=0.0.0.0**

### 二、MHA架构实现
#### 1、免认证登录(两两免认证)

```shell
ssh-keygen
# centos实现
ssh-copu-id -i /root/.ssh/id_rsa '-p 22 root@192.168.1.120' #即可免密登录到192.168.1.120
# ubuntu实现
scp /root/.ssh/id_rsa.pub slave1@192.168.1.120:/home/slave1

cat /home/slave1/id_rsa.pub >> /root/.ssh/authorized_keys # 登录到192.168.1.120上操作

/etc/init.d/ssh restart

```

#### 2、MHA安装配置
[https://www.wandouip.com/t5i124356/](https://www.wandouip.com/t5i124356/ "https://www.wandouip.com/t5i124356/")