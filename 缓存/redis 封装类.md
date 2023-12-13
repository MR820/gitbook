```php
<?php
class redis_storage {
	/**
	 * @var Redis
	 */
	private $link = null;
	public function __construct($host, $port, $auth = '', $timeout = 3) {
		$this->link = new Redis();
		if ($this->link->pconnect($host, $port, $timeout) == false) {
			  throw new Exception('无法连接redis服务器:'.$host.'端口:'.$port, 600);
		}

		if ($auth != '') {
			if ($this->link->auth($auth) == false) {
				throw new Exception('redis服务器登陆认证失败', 601);
			}
		}
	}

	public function set($key, $val, $ttl = 7200) {
		return $this->link->setex($key, $ttl, $val);
	}

	public function get($key) {
		return $this->link->get($key);
	}

	public function delete($key) {
		return $this->link->delete($key);
	}

	/**
	 * 判断指定的key是否存在
	 * @param string $key
	 * @return boolean
	 */
	public function is_exits($key){
		return $this->link->exists($key);
	}

	/**
	 * 向redis 队列写入一条数据
	 * @param string $key 队列名
	 * @param String $data 队列数据
	 * Return true||false
	 */
	public function pushData($key,$data)
	{
		return $this->link->lPush($key, $data);
	}

	/**
	 * 从队列读取一条数据
	 */
	public function popData($key)
	{
		return $this->link->brPop($key , 5); // 5s timeout
	}

	public function ping() {
		return $this->link->ping();
	}

	public function getLink() {
		return $this->link;
	}

	public function __destruct() {
	}
}
```

```php
<?php

/**
 * 淘宝缓存数据库
 */
class redis_taobao
{

    /**
     *
     * @var redis_storage
     */
    private static $db = array();

    private static function getInterface($port)
    {
        if (is_object(self::$db[$port]))
            return self::$db[$port];

        $port = $port ? $port: '9400';
        self::$db[$port] = new redis_storage(YD_REDIS_TAOBAO_HOST, $port, YD_REDIS_TAOBAO_AUTH, YD_REDIS_TAOBAO_TIMEOUT);
        return self::$db[$port];
    }

    public static function __callStatic($name, $args)
    {
        $callback = array(
            self::getInterface(),
            $name
        );
        return call_user_func_array($callback, $args);
    }

    /**
     * 获取redis连接对象
     */
    public static function getDb($port)
    {
        return self::getInterface($port);
    }
}

```