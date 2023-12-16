```php
<?php

class db_base
{

    /**
     *
     * @var PDO
     */
    private $link = null;

    private $statement = null;
    // 记录执行过的SQL语句.
    private $sql = array();

    public function __construct($dbhost, $dbuser, $dbpass, $dbname ,$dbport=3306 )
    {
        //$dsn = 'mysql:dbname=' . $dbname . ';host=' . $dbhost;
		$dsn = 'mysql:dbname=' . $dbname .';host=' . $dbhost .';port='.$dbport;
        $options = array(
            PDO::ATTR_TIMEOUT => 3,
            PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8'
        );

        try {
            $this->link = new PDO($dsn, $dbuser, $dbpass, $options);
            $this->link->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (PDOException $e) {
            throw new Exception($dbhost.$e->getMessage(), 500);
        }
    }

    public function fetchOne($query, $input_parameters = array(), $type = PDO::FETCH_ASSOC)
    {
        try {
            $this->statement = $this->query($query);
            if (is_object($this->statement)) {
                $this->statement->execute($input_parameters);
                $data = $this->statement->fetch($type);
                $this->statement->closeCursor();
                return $data;
            }
        } catch (PDOException $e) {
            is_object($this->statement) && $this->statement->closeCursor();
            throw new Exception($e->getMessage(), 501);
        }
        return false;
    }

    public function fetchAll($query, $input_parameters = array(), $type = PDO::FETCH_ASSOC)
    {
        try {
            $this->statement = $this->query($query);
            if (is_object($this->statement)) {
                $this->statement->execute($input_parameters);
                $data = $this->statement->fetchAll($type);
                $this->statement->closeCursor();
                return $data;
            }
        } catch (PDOException $e) {
            is_object($this->statement) && $this->statement->closeCursor();
            throw new Exception($e->getMessage(), 502);
        }
        return false;
    }

    public function query($sql, $params = array())
    {
        $this->sql[] = $sql;
        try {
            $sql_type = strtoupper(substr(ltrim($sql), 0, 6));
            if (in_array($sql_type, array(
                'INSERT',
                'UPDATE',
                'DELETE'
            ))) {
                $this->statement = $this->link->prepare($sql);
                $response = $this->statement->execute($params);
                $this->statement->closeCursor();
                return $response;
            } else {
                return $this->link->prepare($sql);
            }
        } catch (PDOException $e) {
            throw new Exception($e->getMessage(), 503);
        }
    }

    /**
     * 获取PDO对象.
     * return PDO
     */
    public function getLink()
    {
        return $this->link;
    }

    public function lastInsertId()
    {
        return $this->link->lastInsertId();
    }

    public function quote($string)
    {
        return $this->link->quote($string);
    }

    public function getSql()
    {
        return $this->sql;
    }

    public function ping()
    {
        return is_object($this->link) ? true : false;
    }

    public function close()
    {
        is_object($this->statement) && $this->statement->closeCursor();
        $this->link = null;
    }

    public function __destruct()
    {
        $this->close();
    }
}

```

```php
<?php

class db_ydserver
{

    /**
     *
     * @var Database
     */
    private static $db = null;

    private static function getInterface()
    {
        if (is_object(self::$db))
            return self::$db;

        self::$db = new db_base(OOS_DB_HOST_YDSERVER, OOS_DB_USER_YDSERVER, OOS_DB_PASS_YDSERVER, OOS_DB_NAME_YDSERVER, OOS_DB_PORT_YDSERVER);
        return self::$db;
    }

    public static function __callStatic($name, $args)
    {
        $callback = array(
            self::getInterface(),
            $name
        );
        return call_user_func_array($callback, $args);
    }
}

```