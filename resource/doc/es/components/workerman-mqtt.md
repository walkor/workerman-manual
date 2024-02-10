# workerman/mqtt

MQTT es un protocolo de transporte de mensajes en modo de publicación/suscripción con una arquitectura cliente-servidor que se ha convertido en una parte importante de Internet de las cosas. Su diseño se caracteriza por ser ligero, abierto, simple y estándar, y es fácil de implementar. Estas características lo convierten en una buena elección para muchos escenarios, especialmente para entornos restringidos como la comunicación máquina a máquina (M2M) y el Internet de las cosas (IoT).

workerman\mqtt es una biblioteca de cliente MQTT asincrónico basada en workerman que se puede utilizar para recibir o enviar mensajes con el protocolo MQTT. Es compatible con `QoS 0`, `QoS 1` y `QoS 2`. Admite las versiones `MQTT` `3.1`, `3.1.1` y `5`.

# Repositorio
https://github.com/walkor/mqtt

# Instalación 
```bash
composer require workerman/mqtt
```

# Ejemplo
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
Ejecutar en la línea de comandos con  ```php subscribe.php start``` para iniciar.

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

Ejecutar en la línea de comandos con ```php publish.php start``` para iniciar.

## Interfaz de workerman\mqtt\Client

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

Crea una instancia de cliente MQTT.

  * `$address` es la dirección del servidor MQTT en formato similar a 'mqtt://test.mosquitto.org:1883'. 
  * `$options` es un array de opciones del cliente, que puede configurar las siguientes opciones:
    * `keepalive`: Intervalo de tiempo en segundos en el que el cliente envía un mensaje de latido al servidor, por defecto es 50 segundos, configurado en 0 significa desactivar el latido.
    * `client_id`: ID del cliente, si no se establece, el valor predeterminado es ```"workerman-mqtt-client-".mt_rand()```
    * `protocol_name`: Nombre del protocolo, `MQTT` (versión 3.1.1) o `MQIsdp` (versión 3.1), por defecto es `MQTT`
    * `protocol_level`: Nivel del protocolo, con `protocol_name` como `MQTT` el valor es `4`, con `protocol_name` como `MQIsdp` el valor es `3`
    * `clean_session`: Sesión limpia, por defecto es `true`. Configurado como `false` permite recibir mensajes sin conexión con niveles `QoS 1` y `QoS 2`
    * `reconnect_period`: Intervalo de tiempo para reconexión, por defecto es `1` segundo, `0` significa no reconexión
    * `connect_timeout`: Tiempo de espera de conexión MQTT, por defecto es `30` segundos
    * `username`: Nombre de usuario, opcional
    * `password`: Contraseña, opcional
    * `will`: Mensaje de voluntad, cuando el cliente se desconecta, el broker envía automáticamente un mensaje de voluntad a otros clientes. El formato es:
      * `topic`: Tema
      * `content`: Contenido
      * `qos`: Nivel de `QoS`
      * `retain`: Marcar para retener
    * `resubscribe`: Cuando la conexión se desconecta de forma anormal y se vuelve a conectar, ¿volver a suscribirse a los temas anteriores?, por defecto es `true`
    * `bindto`: Se utiliza para especificar a qué IP y puerto local debe conectarse el cliente al broker, el valor predeterminado es ''
    * `ssl`: Opción SSL, por defecto es `false`, si se establece como `true`, se conecta utilizando SSL. También admite pasar un array de contexto SSL para configurar certificados locales, etc., el contexto SSL se refiere a https://php.net/manual/en/context.ssl.php
    * `debug`: ¿Activar el modo de depuración?, el modo de depuración puede generar información detallada sobre la comunicación con el broker, por defecto es `false`

-------------------------------------------------------

<a name="connect"></a>
### connect()

Conectarse al broker.

-------------------------------------------------------

<a name="publish"></a>
### publish(String $topic, String $content, [array $options], [callable $callback])

Publica un mensaje en un tema dado.

* `$topic` es el tema
* `$content` es el mensaje
* `$options` es un array de opciones que incluye:
  * `qos`: Nivel de `QoS`, por defecto es `0`
  * `retain`: Marcar para retener, por defecto es `false`
  * `dup`: Bandera de repetición, por defecto es `false`
* `$callback` - `function (\Exception $exception = null)` (no compatible con `QoS 0`) Se activa cuando se produce un error o cuando se publica con éxito, `$exception` es el objeto de excepción, cuando no hay errores, `$exception` es `null`, lo mismo para las siguientes funciones.

-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $topic, [array $options], [callable $callback])

Suscribe a uno o varios temas.

* `$topic` es una cadena (suscribirse a un tema) o un array (suscribirse a varios temas) 
  cuando se suscribe a varios temas, `$topic` es un array de temas y niveles de `QoS`, por ejemplo `array('tema1'=> 0, 'tema2'=> 1)`
* `$options` es un array que incluye:
  * `qos`: Nivel de `QoS`, por defecto es `0`
* `$callback` - `function (\Exception $exception = null, array $granted = [])`)
  Función de callback, se activa cuando la suscripción es exitosa o cuando se produce un error.
  * `exception`: Objeto de excepción, es `null` si no hay errores.
  * `granted`: Array de resultados de la suscripción, similar a `array('tema' => 'qos', 'tema' => 'qos')` donde:
    * `tema` es el tema suscrito
    * `qos` es el nivel de `QoS` aceptado por el broker.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $topic, [callable $callback])

Cancelar la suscripción a uno o varios temas.

* `$topic` es una cadena o un array de cadenas, similar a `array('tema1', 'tema2')`.
* `$callback` - `function (\Exception $e = null)`, se activa cuando la operación tiene éxito o falla.

-------------------------------------------------------

<a name="disconnect"></a>
### disconnect()

Desconectarse del broker de manera normal, se enviará un mensaje `DISCONNECT` al broker.

-------------------------------------------------------

<a name="close"></a>
### close()

Desconectarse del broker de manera forzada, no se enviará un mensaje `DISCONNECT` al broker.

-------------------------------------------------------

<a name="onConnect"></a>
### callback onConnect(Client $mqtt)

Se activa cuando se establece la conexión con el broker. En este punto, ya se ha recibido el mensaje `CONNACK` del broker.

-------------------------------------------------------

<a name="onMessage"></a>
### callback onMessage(String $topic, String $content, Client $mqtt)

Se activa cuando el cliente recibe un mensaje de publicación.
* `$topic` es el tema recibido
* `$content` es el contenido del mensaje recibido
* `$mqtt` es la instancia del cliente MQTT.

-------------------------------------------------------

<a name="onError"></a>
### callback onError(\Exception $exception = null)

Se activa cuando se produce un error en la conexión.

-------------------------------------------------------

<a name="onClose"></a>
### callback onClose()

Se activa cuando se cierra la conexión, ya sea porque el cliente la cerró voluntariamente o porque el servidor la cerró.
