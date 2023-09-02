---
title: php的Pdo扩展实现类似mysql_ping的方法
author: admin
type: post
date: 2017-10-17T06:18:33+00:00
url: /archives/17475
categories:
 - 程序开发
tags:
 - php

---
在php里Pdo是没有mysql\_ping和mysqli\_ping函数的，可以使用以下方法来代替它

```
class NPDO {
    private $pdo;
    private $params;

    public function __construct() {
        $this->params = func_get_args();
        $this->init();
    }

    public function __call($name, array $args) {
        return call_user_func_array(array($this->pdo, $name), $args);
    }

    // The ping() will try to reconnect once if connection lost.
    public function ping() {
        try {
            $this->pdo->query('SELECT 1');
        } catch (PDOException $e) {
            $this->init();            // Don't catch exception here, so that re-connect fail will throw exception
        }

        return true;
    }

    private function init() {
        $class = new ReflectionClass('PDO');
        $this->pdo = $class->newInstanceArgs($this->params);
    }
}

```

转自： [https://terenceyim.wordpress.com/2009/01/09/adding-ping-function-to-pdo/](https://terenceyim.wordpress.com/2009/01/09/adding-ping-function-to-pdo/)

或者使用以下方法（目前本人在用）

```
/**
     * 检查连接是否可用
     * @param  Link $dbconn 数据库连接
     * @return Boolean
     */
    function ping() {
        try {
            $ret = $this->getAttribute(constant("PDO::ATTR_SERVER_INFO"));
            if ($ret === null) {
                return false;
            }

        } catch (\PDOException $e) {
            if(strpos($e->getMessage(), 'MySQL server has gone away')!==false){
                echo PHP_EOL . $e->getMessage() . PHP_EOL;
                return false;
            }
        }
        return true;
    }

```