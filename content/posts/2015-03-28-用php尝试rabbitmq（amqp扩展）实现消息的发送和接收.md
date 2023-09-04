---
title: 用PHP尝试RabbitMQ（amqp扩展）实现消息的发送和接收
author: admin
type: post
date: 2015-03-28T14:10:23+00:00
url: /archives/15491
categories:
 - 系统架构
tags:
 - amqp
 - RabbitMQ

---
上篇文章我们介绍了 [amqp扩展在windows下的安装方法](http://blog.haohtml.com/archives/15484)，这里我们看一下用法。

**消费者：接收消息**

逻辑：
创建连接–>创建channel–>创建交换机–>创建队列–>绑定交换机/队列/路由键–>接收消息

```
<?php
/*************************************
* PHP amqp(RabbitMQ) Demo - consumer
* Author: Linvo
* Date: 2012/7/30
*************************************/
//配置信息
$conn_args = array(
    'host' => '192.168.1.93',
    'port' => '5672',
    'login' => 'guest',
    'password' => 'guest',
    'vhost'=>'/'
);
$e_name = 'e_linvo'; //交换机名
$q_name = 'q_linvo'; //队列名
$k_route = 'key_1'; //路由key

//创建连接和channel
$conn = new AMQPConnection($conn_args);
if (!$conn->connect()) {
    die("Cannot connect to the broker!\n");
}
$channel = new AMQPChannel($conn);

//创建交换机
$ex = new AMQPExchange($channel);
$ex->setName($e_name);
$ex->setType(AMQP_EX_TYPE_DIRECT); //direct类型
$ex->setFlags(AMQP_DURABLE); //持久化
echo "Exchange Status:".$ex->declare()."\n";

//创建队列
$q = new AMQPQueue($channel);
$q->setName($q_name);
$q->setFlags(AMQP_DURABLE); //持久化
echo "Message Total:".$q->declare()."\n";

//绑定交换机与队列，并指定路由键
echo 'Queue Bind: '.$q->bind($e_name, $k_route)."\n";

//阻塞模式接收消息
echo "Message:\n";
while(True){
    $q->consume('processMessage');
    //$q->consume('processMessage', AMQP_AUTOACK); //自动ACK应答
}
$conn->disconnect();

/**
* 消费回调函数
* 处理消息
*/
function processMessage($envelope, $queue) {
    $msg = $envelope->getBody();
    echo $msg."\n"; //处理消息
    $queue->ack($envelope->getDeliveryTag()); //手动发送ACK应答
}
```

**生产者：发送消息**

逻辑：
创建连接–>创建channel–>创建交换机对象–>发送消息

```
<?php
/*************************************
* PHP amqp(RabbitMQ) Demo - publisher
* Author: Linvo
* Date: 2012/7/30
*************************************/
//配置信息
$conn_args = array(
    'host' => '192.168.1.93',
    'port' => '5672',
    'login' => 'guest',
    'password' => 'guest',
    'vhost'=>'/'
);
$e_name = 'e_linvo'; //交换机名
//$q_name = 'q_linvo'; //无需队列名
$k_route = 'key_1'; //路由key

//创建连接和channel
$conn = new AMQPConnection($conn_args);
if (!$conn->connect()) {
    die("Cannot connect to the broker!\n");
}
$channel = new AMQPChannel($conn);

//消息内容
$message = "TEST MESSAGE! 测试消息！";

//创建交换机对象
$ex = new AMQPExchange($channel);
$ex->setName($e_name);

//发送消息
//$channel->startTransaction(); //开始事务
for($i=0; $i<5; ++$i){
    echo "Send Message:".$ex->publish($message, $k_route)."\n";
}
//$channel->commitTransaction(); //提交事务

$conn->disconnect();
```

需要注意的地方是：

queue对象有两个方法可用于取消息：consume和get。
前者是阻塞的，无消息时会被挂起，适合循环中使用；

后者则是非阻塞的，取消息时有则取，无则返回false。

**测试截图**

运行消费者：

![1343873405_2469](http://blogx.haohtml.com/wp-content/uploads/2014/03/1343873405_2469.jpg)

运行生产者，发消息：

![1343873408_3083](http://blogx.haohtml.com/wp-content/uploads/2014/03/1343873408_3083.jpg)

消费者接收到消息：

![1343873412_6992](http://7u2fpb.com1.z0.glb.clouddn.com/wp-content/uploads/2014/03/1343873412_6992.jpg)

转：http://nonfu.me/p/9722.html

更多教程参考： [http://www.rabbitmq.com/getstarted.html](http://www.rabbitmq.com/getstarted.html)