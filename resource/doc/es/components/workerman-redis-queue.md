# workerman/redis-queue

Cola de mensajes basada en Redis, con soporte para el procesamiento de mensajes con retraso.

## Dirección del proyecto:
https://github.com/walkor/redis-queue

## Instalación:
```composer require workerman/redis-queue```

## Ejemplo
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Suscribirse
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Suscribirse
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Enviar mensajes a la cola en intervalos regulares
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## API
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

Crear una instancia

  * `$address`  similar a `redis://ip:6379`, debe comenzar con redis. 

  * `$options`  incluye las siguientes opciones:
    * `auth`: información de autenticación, por defecto ''
    * `db`: base de datos, por defecto 0
    * `max_attempts`: número de intentos de reintentos después de un consumo fallido, por defecto 5
    * `retry_seconds`: intervalo de tiempo entre reintentos, en segundos. Por defecto 5

> El consumo fallido se refiere a cuando el negocio arroja una excepción `Exception` o `Error`. Después de un consumo fallido, el mensaje se coloca en la cola de retraso para volver a intentarlo, el número de intentos se controla con `max_attempts`, y el intervalo de reintentos se controla a través de `retry_seconds` y `max_attempts`. Por ejemplo, si `max_attempts` es 5 y `retry_seconds` es 10, el intervalo de tiempo para el primer reintentoes `1*10` segundos, para el segundo es `2*10` segundos, para el tercero es `3*10` segundos, y así sucesivamente hasta 5 intentos de reintentos. Si se supera el número de intentos configurado en `max_attempts`, el mensaje se coloca en la cola de fallidos con la clave `{redis-queue}-failed` (antes de la versión 1.0.5 era `redis-queue-failed`)

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Enviar un mensaje a la cola

* `$queue` nombre de la cola, tipo `String`
* `$data` mensaje específico a publicar, puede ser un array o una cadena, tipo `Mixed`
* `$dely` tiempo de retraso en el consumo, en segundos, por defecto 0, tipo `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Suscribirse a una o varias colas

* `$queue` nombre de la cola, puede ser una cadena o un array que contiene varios nombres de cola
* `$callback` función de devolución de llamada, con el formato `function (Mixed $data)`, donde `$data` es el mismo que `$data` en `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Cancelar la suscripción

* `$queue` nombre de la cola o un array que contiene varios nombres de cola

-------------------------------------------------------

## Enviar mensajes a la cola en un entorno no workerman
A veces, algunos proyectos se ejecutan en entornos como apache o php-fpm, donde no se puede utilizar el proyecto workerman/redis-queue. En ese caso, se puede utilizar la siguiente función para enviarlos:
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //antes de la versión 1.0.5 era redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//antes de la versión 1.0.5 era redis-queue-delayed
    
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```
Donde el parámetro `$redis` es una instancia de redis. Por ejemplo, el uso de la extensión redis es similar al siguiente ejemplo:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
