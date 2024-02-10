# workerman/mqtt
MQTT - это протокол передачи сообщений в режиме издатель/подписчик с клиент-серверной архитектурой, который стал важной частью интернета вещей. Его дизайн легок, открыт, прост и стандартизирован, что облегчает его реализацию. Эти особенности делают его хорошим выбором для многих сценариев, особенно для коммуникации между ограниченными устройствами (M2M) и средой интернета вещей (IoT).

workerman\mqtt - это асинхронная клиентская библиотека MQTT, основанная на workerman, которая может быть использована для приема или отправки сообщений по протоколу MQTT. Поддерживает `QoS 0`, `QoS 1`, `QoS 2`. Поддерживает версии `MQTT` `3.1`, `3.1.1`, `5`.

# Ссылка на проект
https://github.com/walkor/mqtt

# Установка
```php
composer require workerman/mqtt
```

# Пример
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
Запуск в командной строке: ```php subscribe.php start``` для запуска.

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
Запуск в командной строке: ```php publish.php start``` для запуска.

## Интерфейс workerman\mqtt\Client

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

Создает экземпляр клиента mqtt.

  * `$address` - адрес сервера mqtt в формате 'mqtt://test.mosquitto.org:1883'.
  * `$options` - массив опций клиента, в котором можно установить следующие параметры:
    * `keepalive`: интервал времени между отправкой клиентом сердечных импульсов на сервер, по умолчанию 50 секунд, установка в 0 означает отключение сердечных импульсов
    * `client_id`: идентификатор клиента, если не установлен, то по умолчанию ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: название протокола, `MQTT` (версия 3.1.1) или `MQIsdp` (версия 3.1), по умолчанию `MQTT`
    * `protocol_level`: уровень протокола, равен `4` при `protocol_name` равном `MQTT`, и равен `3` при `protocol_name` равном `MQIsdp`
    * `clean_session`: очистка сеанса, по умолчанию `true`. Установка в `false` позволяет принимать оффлайн-сообщения уровня `QoS 1` и `QoS 2`
    * `reconnect_period`: интервал времени для повторного соединения, по умолчанию `1` секунда, `0` означает отключение повторного соединения
    * `connect_timeout`: время ожидания соединения mqtt, по умолчанию `30` секунд
    * `username`: имя пользователя, необязательно
    * `password`: пароль, необязательно
    * `will`: сообщение о воле, когда клиент отключается, Брокер автоматически отправляет сообщение о воле другим клиентам. Формат:
      * `topic`: тема
      * `content`: содержание
      * `qos`: уровень `QoS`
      * `retain`: метка `retain`
    * `resubscribe`: при обрыве соединения и повторном подключении, повторно подписываться на предыдущие темы, по умолчанию `true`
    * `bindto`: указывает, через какой IP и порт создать соединение с Брокером, по умолчанию ''
    * `ssl`: настройки ssl, по умолчанию `false`, если установлено в `true`, то соединение происходит через ssl. Также поддерживает передачу массива контекста ssl для настройки локальных сертификатов и т. д., см. контекст ssl: https://php.net/manual/en/context.ssl.php
    * `debug`: включить или отключить режим отладки, по умолчанию `false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

Соединение с Брокером

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Публикация сообщения в определенную тему

* `$topic` - тема
* `$content` - сообщение
* `$options` - массив опций, включает
  * `qos` - уровень `QoS`, по умолчанию `0`
  * `retain` - метка `retain`, по умолчанию `false`
  * `dup` - флаг повторной отправки, по умолчанию `false`
* `$callback` - `function (\Exception $exception = null)` (не поддерживается для `QoS 0`) - срабатывает при возникновении ошибки или успешной публикации, `$exception` является объектом исключения, когда ошибок нет, `$exception` равен `null`, аналогично ниже.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

Подписка на одну или несколько тем

* `$topic` - строка (подписаться на одну тему) или массив (подписаться на несколько тем), 
при подписке на несколько тем, `$topic` представляет собой массив, в котором ключ - тема, значение - `QoS`, например, `array('topic1'=> 0, 'topic2'=> 1)`
* `$options` - массив параметров подписки, включая следующие настройки:
  * `qos` - уровень `QoS`, по умолчанию `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`
  Функция обратного вызова, срабатывает при успешном выполнении подписки или при возникновении ошибок
  * `exception` - объект исключения, если ошибок нет, то он равен `null`
  * `granted` - массив результатов подписки, похожий на `array('topic' => 'qos', 'topic' => 'qos')`, где:
    * `topic` - подписанная тема
    * `qos` - уровень `QoS`, принятый Брокером

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Отписка

* `$topic` - строка или массив строк, например, `array('topic1', 'topic2')`
* `$callback` - `function (\Exception $e = null)`, срабатывает при успешном или неудачном выполнении

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Нормальное отключение соединения с Брокером, отправляется сообщение `DISCONNECT` на Брокер.

-------------------------------------------------------

<a name="close"></a>
### close()

Принудительное отключение от Брокера, сообщение `DISCONNECT` не отправляется Брокеру.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)
Срабатывает после установления соединения с Брокером. В этот момент уже было получено сообщение `CONNACK` от Брокера.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)
`function (topic, message, packet) {}`

Срабатывает при получении клиентом сообщения Publish
* `$topic` - полученная тема
* `$content` - содержание полученного сообщения
* `$mqtt` - экземпляр клиента mqtt.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)
Срабатывает при возникновении ошибки соединения

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()
Срабатывает при закрытии соединения, независимо от того, закрыл ли клиент соединение самостоятельно или сервер закрыл соединение.
