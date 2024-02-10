# Ejemplo simple de desarrollo

## Instalación

**Instalar workerman**
Ejecute en un directorio vacío
`composer require workerman/workerman`

## Ejemplo uno, proporcionar servicios web al externo utilizando el protocolo HTTP
**Cree el archivo start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Crea un Worker que escucha en el puerto 2345, utilizando el protocolo http
$http_worker = new Worker("http://0.0.0.0:2345");

// Inicia 4 procesos para proporcionar servicios externos
$http_worker->count = 4;

// Responde con "hello world" al recibir datos del navegador
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Envía "hello world" al navegador
    $connection->send('hello world');
};

// Ejecuta el Worker
Worker::runAll();
```

**Ejecución en la línea de comandos (para usuarios de Windows, use [cmd](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn))**
```shell
php start.php start
```

**Prueba**

Supongamos que la IP del servidor es 127.0.0.1

Acceda a la URL http://127.0.0.1:2345 en un navegador

**Nota:**

1. Si surge algún problema de acceso, consulte la sección [Razones de fallo en la conexión del cliente](../faq/client-connect-fail.md).

2. El servidor utiliza el protocolo HTTP y solo puede comunicarse a través del mismo protocolo, no se puede utilizar directamente para WebSocket u otros protocolos.

## Ejemplo dos, proporcionar servicios externos utilizando el protocolo WebSocket
**Cree el archivo ws_test.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Nota: Aquí, a diferencia del ejemplo anterior, se utiliza el protocolo websocket
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Inicia 4 procesos para proporcionar servicios externos
$ws_worker->count = 4;

// Al recibir datos del cliente, devuelve "hello $data" al cliente
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Envía "hello $data" al cliente
    $connection->send('hello ' . $data);
};

// Ejecuta el Worker
Worker::runAll();
```

**Ejecución en la línea de comandos**
```shell
php ws_test.php start
```

**Prueba**

Abra el navegador Chrome, presione F12 para abrir la consola de depuración. En la pestaña Console, ingrese (o agregue el siguiente código a una página HTML y ejecútelo con JavaScript)

```javascript
// Suponga que la IP del servidor es 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Conexión exitosa");
    ws.send('tom');
    alert("Enviar una cadena al servidor: tom");
};
ws.onmessage = function(e) {
    alert("Mensaje recibido del servidor: " + e.data);
};
```

**Nota:**

1. Si surge algún problema de acceso, consulte la sección [Falla de conexión del cliente](../faq/client-connect-fail.md) en el manual.

2. El servidor utiliza el protocolo WebSocket y solo puede comunicarse a través del mismo protocolo, no se puede utilizar directamente para HTTP u otros protocolos.

## Ejemplo tres, envío directo de datos a través de TCP
**Cree el archivo tcp_test.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Crea un Worker que escucha en el puerto 2347, sin utilizar ningún protocolo de capa de aplicación
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Inicia 4 procesos para proporcionar servicios externos
$tcp_worker->count = 4;

// Cuando el cliente envía datos
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Envía "hello $data" al cliente
    $connection->send('hello ' . $data);
};

// Ejecuta el Worker
Worker::runAll();
```

**Ejecución en la línea de comandos**
```shell
php tcp_test.php start
```

**Prueba: Ejecución en la línea de comandos**
(El efecto en la línea de comandos de Linux a continuación es ligeramente diferente al de Windows)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**Nota:**

1. Si surge algún problema de acceso, consulte la sección [Falla de conexión del cliente](../faq/client-connect-fail.md) en el manual.

2. El servidor utiliza el protocolo TCP sin formato, no puede utilizarse directamente para WebSocket, HTTP y otros protocolos.
