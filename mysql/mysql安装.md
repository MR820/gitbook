## 1、ubuntu 启动、停止、重启服务（以mysql数据库为例）

### 启动mysql：
```shell
sudo /etc/init.d/mysql start

sudo start mysql

sudo service mysql start
```
### 停止mysql:
```shell
sudo /etc/init.d/mysql stop

sudo stop mysql

sudo service mysql stop
```
### 重启myql:
```shell
sudo /etc/init.d/mysql restart

sudo restart msyql

sudo service msyql restart
```
## 2、ubuntu 安装配置mysql 5.7

### 安装
```shell
sudo apt install mysql-server

##sudo apt install mysql-client
```
### 卸载mysql
```shell
sudo apt autoremove --purge mysql-server-5.7

sudo apt remove mysql-server

sudo apt autoremove mysql-server

sudo apt remove mysql-common
```
### 清理残余数据
```shell
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
```
### 检查mysql是否运行
```shell
sudo netstat -tap | grep mysql
```
### 登陆mysql
```shell
sudo cat /etc/mysql/debian.cnf
```
### 设置root密码
```shell
use mysql;

update user set authentication_string=password('password') where user='root' and Host = 'localhost';
```

### mariadb修改密码
```shell
use mysql;

update user set password=password('password') where user='username' and Host='localhost';
```

### 重启mysql
```shell
sudo /etc/init.d/mysql restart
```
## 3、配置远程访问

### 注释bind-address = 127.0.0.1
```shell
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```
### 打开iptables 3306端口
```shell
iptables -I INPUT 4 -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
iptables-save > /etc/iptables.up.rules #保存iptables规则
```
### 数据库授权
```shell
grant all privileges on db_name.* to db_user@'%' identified by 'db_pass';
flush privileges;
```
## 关闭远程访问
### 关闭iptables 3306端口
### 收回权限
```shell
mysql revoke #收回权限
```