# workerman/mqtt
MQTT is a message transmission protocol based on the client-server architecture and the publish/subscribe pattern, which has become an important part of the Internet of Things. Its design philosophy is lightweight, open, simple, and standardized, and easy to implement. These characteristics make it a good choice for many scenarios, especially for constrained environments such as machine-to-machine communication (M2M) and the Internet of Things (IoT).

workerman\mqtt is an asynchronous MQTT client library based on workerman, which can be used to receive or send MQTT protocol messages. It supports `QoS 0`, `QoS 1`, `QoS 2`. It also supports MQTT versions `3.1`, `3.1.1`, and `5`.

# Project Address
https://github.com/walkor/mqtt

# Installation
```bash
composer require workerman/mqtt
```

# Examples
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
Run in the command line  ```php subscribe.php start``` to start.

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
Run in the command line ```php publish.php start``` to start.

## workerman\mqtt\Client Interface

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

Creates an instance of the MQTT client.

  * `$address` is the MQTT server address, in the format 'mqtt://test.mosquitto.org:1883'.

  * `$options` is an array of client options, which can be set as follows:
    * `keepalive`: The interval in seconds for the client to send heartbeats to the server. The default is 50 seconds. Setting it to 0 means disabling heartbeats.
    * `client_id`: The client ID. If not set, it defaults to ```"workerman-mqtt-client-".mt_rand()```.
    * `protocol_name`: The protocol name, either `MQTT` (for version 3.1.1) or `MQIsdp` (for version 3.1). The default is `MQTT`.
    * `protocol_level`: The protocol level. If `protocol_name` is `MQTT`, the value is `4`, if `protocol_name` is `MQIsdp`, the value is `3`.
    * `clean_session`: Whether to clean the session. The default is `true`. Setting it to `false` allows receiving offline messages with `QoS 1` and `QoS 2`.
    * `reconnect_period`: The interval at which to reconnect, in seconds. The default is `1` second. Setting it to `0` means no reconnection.
    * `connect_timeout`: The connection timeout for MQTT, in seconds. The default is `30` seconds.
    * `username`: The username, optional.
    * `password`: The password, optional.
    * `will`: The last will and testament message. When the client disconnects, the Broker will automatically send the last will message to other clients. It has the following format:
      * `topic`: Topic of the message
      * `content`: Content of the message
      * `qos`: `QoS` level
      * `retain`: `retain` flag
    * `resubscribe`: Whether to resubscribe to previous topics after reconnecting in case of abnormal disconnection. The default is true.
    * `bindto`: Specifies the local IP and port to initiate the connection to the Broker. The default value is ''.
    * `ssl`: SSL options. The default is `false`. If set to `true`, it connects via SSL. It also supports passing an array of SSL context settings to configure local certificates, etc. The SSL context reference can be found at https://php.net/manual/en/context.ssl.php.
    * `debug`: Whether to enable debug mode. Debug mode can output detailed information about communication with the Broker. The default is `false`.

-------------------------------------------------------

<a name="connect"></a>
### connect()

Connects to the Broker.

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Publishes a message to a specific topic.

* `$topic`: The topic.
* `$content`: The message.
* `$options`: An array of options, including:
  * `qos`: The `QoS` level. The default is `0`.
  * `retain`: The retain flag. The default is `false`.
  * `dup`: The duplicate delivery flag. The default is `false`.
* `$callback`: `function (\Exception $exception = null)`. (This setting is not supported for QoS 0.) Triggered when an error occurs or when the publication is successful. `$exception` is the exception object, and when there is no error, `$exception` is `null`.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

Subscribes to one or multiple topics.

* `$topic`: A string (for a single topic) or an array (for multiple topics). When subscribing to multiple topics, `$topic` is an associative array with topic as the key and QoS as the value, for example, `array('topic1'=> 0, 'topic2'=> 1)`.
* `$options`: An array of subscription options, including:
  * `qos`: The `QoS` level. The default is `0`.
* `$callback`: `function (\Exception $exception = null, array $granted = [])`. Triggered when the subscription is successful or an error occurs.
  * `exception`: The exception object. When there is no error, it is `null`.
  * `granted`: The array of subscription results, similar to `array('topic' => 'qos', 'topic' => 'qos')`, where:
    * `topic`: The subscribed topic.
    * `qos`: The `QoS` level accepted by the Broker.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Unsubscribes from one or multiple topics.

* `$topic`: A string or an array of strings, for example, `array('topic1', 'topic2')`.
* `$callback`: `function (\Exception $e = null)`. Triggered when the unsubscription is successful or fails.

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Gracefully disconnects from the Broker. A `DISCONNECT` message will be sent to the Broker.

-------------------------------------------------------

<a name="close"></a>
### close()

Forcibly disconnects from the Broker without sending a `DISCONNECT` message.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)

Triggered when the connection to the Broker is established. At this point, the client has received the `CONNACK` message from the Broker.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)

Triggered when the client receives a Publish message.
* `$topic`: The received topic.
* `$content`: The received message content.
* `$mqtt`: The MQTT client instance.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)

Triggered when a connection error occurs.

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()

Triggered when the connection is closed, either by the client or the server.
