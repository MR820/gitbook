### 1、php.ini配置文件
```
session.save_handler = redis
session.save_path = “tcp://127.0.0.1:6379″
```
需改后重启php-fpm

### 2、php代码
```
ini_set(“session.save_handler”,”redis”);
ini_set(“session.save_path”,”tcp://127.0.0.1:6379″);
```

### 3、使用示例
```php
//如果未修改php.ini下面两行注释去掉
//ini_set('session.save_handler', 'redis');
//ini_set('session.save_path', 'tcp://127.0.0.1:6379');
session_start();
$_SESSION['sessionid'] = 'this is session content!';
echo $_SESSION['sessionid'];
echo '';

$redis = new redis();
$redis->connect('127.0.0.1', 6379);
//redis用session_id作为key并且是以string的形式存储
echo $redis->get('PHPREDIS_SESSION:' . session_id());
```