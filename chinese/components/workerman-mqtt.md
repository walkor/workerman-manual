# workerman/mqtt
MQTT是一个客户端服务端架构的发布/订阅模式的消息传输协议，已经成为物联网的重要组成部分。它的设计思想是轻巧、开放、简单、规范，易于实现。这些特点使得它对很多场景来说都是很好的选择，特别是对于受限的环境如机器与机器的通信（M2M）以及物联网环境（IoT）。

workerman\mqtt 是一个基于workerman的异步mqtt 客户端库，可用于接收或者发送mqtt协议的消息。支持`QoS 0`、`QoS 1`、`QoS 2`。支持`MQTT` `3.1` `3.1.1` `5`版本。


# 项目地址
https://github.com/walkor/mqtt

# 安装 
```
composer require workerman/mqtt
```

# 示例
**subscribe.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
        $mqtt->subscribe('test');
    };
    $mqtt->onMessage = function($topic, $content){
        var_dump($topic, $content);
    };
    $mqtt->connect();
};
Worker::runAll();
```
命令行运行  ```php subscribe.php start``` 启动

**publish.php**
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function(){
    $mqtt = new Workerman\Mqtt\Client('mqtt://test.mosquitto.org:1883');
    $mqtt->onConnect = function($mqtt) {
       $mqtt->publish('test', 'hello workerman mqtt');
    };
    $mqtt->connect();
};
Worker::runAll();
```

命令行运行 ```php publish.php start``` 启动。

## workerman\mqtt\Client接口

  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#connect"><code>Client::<b>connect()</b></code></a>
  * <a href="#publish"><code>Client::<b>publish()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>
  * <a href="#disconnect"><code>Client::<b>disconnect()</b></code></a>
  * <a href="#close"><code>Client::<b>close()</b></code></a>
  * <a href="#onConnect"><code>callback <b>onConnect</b></code></a>
  * <a href="#onMessage"><code>callback <b>onMessage</b></code></a>
  * <a href="#onError"><code>callback <b>onError</b></code></a>
  * <a href="#onClose"><code>callback <b>onClose</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

创建一个mqtt客户端实例。

  * `$address` mqtt服务端地址，格式类似 'mqtt://test.mosquitto.org:1883'. 

  * `$options` 客户端选项数组，可以设置以下选项：
    * `keepalive`: 客户端向服务端发送心跳的时间间隔，默认50秒，设置成0代表不启用心跳
    * `client_id`: 客户端id，如果没设置默认是 ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: 协议名，`MQTT`(3.1.1版本)或者 `MQIsdp`(3.1版本)，默认是`MQTT`
    * `protocol_level`:协议等级， `protocol_name`是`MQTT` 时值为`4` ，`protocol_name`是`MQIsdp` 时值是 `3`
    * `clean_session`: 清理会话，默认为`true`。设置为`false`可以接收到`QoS 1`和`QoS 2`级别的离线消息
    * `reconnect_period`: 重连时间间隔，默认 `1` 秒，`0`代表不重连
    * `connect_timeout`: 连接mqtt超时时间，默认`30` 秒
    * `username`: 用户名，可选
    * `password`: 密码，可选
    * `will`: 遗嘱消息，当客户端断线后Broker会自动发送遗嘱消息给其它客户端. 格式为:
      * `topic`: 主题
      * `content`: 内容
      * `qos`: `QoS`等级
      * `retain`: `retain`标记
    * `resubscribe` : 当连接异常断开并重连后，是否重新订阅之前的主题，默认为true
    * `bindto` 用来指定本地以哪个ip和端口向Broker发起连接，默认值为''
    * `ssl` ssl选项，默认是 `false`，如果设置为`true`，则以ssl方式连接。同时支持传入ssl上下文数组，用来配置本地证书等，ssl上下文参考https://php.net/manual/en/context.ssl.php
    * `debug` 是否开启debug模式，debug模式可以输出与Broker通讯的详细信息，默认为`false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

连接Broker

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

向某个主题发布一条消息

* `$topic` 主题
* `$message` 消息
* `$options` 选项数组，包括
  * `qos` `QoS`等级，默认`0`
  * `retain` retain 标记，默认`false`
  * `dup` 重发标志，默认`false`
* `$callback` - `function (\Exception $exception = null)`，（QoS为0时不支持此设置）当发生错误时或者发布成功时触发， `$exception` 是异常对象，当没有错误发生时 `$exception` 为`null`，下同。
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

订阅一个主题或者多个主题

* `$topic` 是一个字符串(订阅一个主题)或者数组(订阅多个主题)，
当订阅多个主题时，`$topic`是主题是key，`QoS`为值的数组，例如`array('topic1'=> 0, 'topic2'=> 1)`
* `$options` 订阅选项数组，包含以下设置:
  * `qos` `QoS`等级, 默认 `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
  回调函数，当订阅成功或者发生错误时触发
  * `exception` 异常对象，无错误发生时它是`null`，下同
  * `granted` 订阅结果数组，类似 `array('topic' => 'qos', 'topic' => 'qos')` 其中:
    * `topic` 是订阅的主题
    * `qos` Broker接受的`QoS`等级

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

取消订阅

* `$topic` 是一个字符串或者字符串数组，类似`array('topic1', 'topic2')`
* `$callback` - `function (\Exception $e = null)`, 成功或者失败时触发的回调

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

正常断开与Broker的连接， `DISCONNECT`报文会被发送到Broker.

-------------------------------------------------------

<a name="close"></a>
### close()

强制断开与Broker的连接，不会发送`DISCONNECT`报文给Broker.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
当与Broker连接建立完毕后触发。这时候已经收到了Broker的`CONNACK` 报文。

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (topic, message, packet) {}`

当客户端收到Publish报文时触发
* `$topic` 收到的主题
* `$content` 收到的消息内容
* `$mqtt` mqtt客户端实例.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
当连接发生某种错误时触发

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
当连接关闭时触发，无论是客户端主动关闭还是服务端关闭都会触发`onClose`


