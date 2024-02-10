# workerman/mqtt
O MQTT é um protocolo de transporte de mensagens no estilo publish/subscribe de arquitetura cliente/servidor e tornou-se uma parte importante da Internet das Coisas (IoT). Sua filosofia de design é de ser leve, aberto, simples, padronizado e fácil de implementar. Essas características fazem com que seja uma ótima escolha para muitos cenários, especialmente para comunicação entre máquinas (M2M) e ambientes de IoT.

workerman\mqtt é uma biblioteca cliente MQTT assíncrona baseada em workerman, que pode ser usada para receber ou enviar mensagens no protocolo MQTT. Suporta `QoS 0`, `QoS 1` e `QoS 2`. Suporta as versões `MQTT` `3.1`, `3.1.1` e `5`.

# Endereço do Projeto
https://github.com/walkor/mqtt

# Instalação
```
composer require workerman/mqtt
```

# Exemplo
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
Execute no terminal ```php subscribe.php start``` para iniciar.

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
Execute no terminal ```php publish.php start``` para iniciar.

## Interface Client de workerman\mqtt

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
