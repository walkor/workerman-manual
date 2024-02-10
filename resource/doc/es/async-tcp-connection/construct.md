# Método __construct
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Crea un objeto de conexión asíncrona.

AsyncTcpConnection permite que Workerman actúe como cliente para establecer una conexión asíncrona con un servidor remoto, y enviar y recibir datos de manera asíncrona a través de la interfaz send y el callback onMessage.

## Parámetros
Parámetro: ``` remote_address ```

La dirección de la conexión, por ejemplo: 
``` tcp://www.baidu.com:80 ```
``` ssl://www.baidu.com:443 ```
``` ws://echo.websocket.org:80 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

Parámetro: ``` $context_option ```

``` Este parámetro es requerido (workerman >= 3.3.5) ```

Usado para establecer el contexto del socket, como utilizar ``` bindto ``` para establecer desde qué (tarjeta de red) dirección IP y puerto se accede a la red externa, configurar certificados SSL, etc.

Consulte [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Opciones de contexto de socket](https://php.net/manual/zh/context.socket.php), [Opciones de contexto de SSL](https://php.net/manual/zh/context.ssl.php)

## Nota
Actualmente, AsyncTcpConnection admite protocolos como [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md).

También admite la personalización de protocolos, consulte [Cómo personalizar protocolos](../protocols/how-protocols.md).

El protocolo [ssl](https://baike.baidu.com/view/525499.htm) requiere Workerman >= 3.3.4, y la extensión de [openssl](https://php.net/manual/zh/book.openssl.php) debe estar instalada.

Actualmente, AsyncTcpConnection no admite el protocolo [http](https://baike.baidu.com/view/9472.htm).

Se puede utilizar ``` new AsyncTcpConnection('ws://...') ``` para iniciar una conexión de websocket con un servidor remoto, al igual que en un navegador, consulte [Ejemplo](../appendices/about-ws.md). Sin embargo, no se puede usar ``` new AsyncTcpConnection('websocket://...') ``` para iniciar una conexión websocket en Workerman.

## Ejemplos
### Ejemplo 1: Acceso asíncrono a un servicio http externo
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Al iniciar el proceso, se establece de manera asíncrona una conexión a www.baidu.com y se envían datos para recibir la respuesta
$task->onWorkerStart = function($task)
{
    // No es posible especificar http directamente, pero se puede simular el protocolo http utilizando tcp para enviar datos
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // Cuando la conexión se establece con éxito, se envía la solicitud http
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexión exitosa\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexión cerrada\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Código de error: $code,  Mensaje: $msg\n";
    };
    $connection_to_baidu->connect();
};

// Ejecutar el worker
Worker::runAll();
```
### Ejemplo 2: Acceso asíncrono a un servicio de websocket externo, y configuración de la dirección IP local y puerto utilizados
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Configurar la dirección IP local y puerto para acceder al host remoto (cada conexión socket ocupará un puerto local)
    $context_option = array(
        'socket' => array(
            // La IP debe ser la IP de la tarjeta de red local y debe poder acceder al host remoto, de lo contrario será inválida
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
### Ejemplo 3: Acceso asíncrono a un puerto wss externo, y configuración de un certificado SSL local
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Configurar la dirección IP local y puerto para acceder al host remoto, y el certificado SSL local
    $context_option = array(
        'socket' => array(
            // La IP debe ser la IP de la tarjeta de red local y debe poder acceder al host remoto, de lo contrario será inválida
            'bindto' => '114.215.84.87:2333',
        ),
        // Opciones ssl, consulte https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Ruta del certificado local. Debe estar en formato PEM e incluir el certificado local y la clave privada.
            'local_cert'        => '/su/ruta/a/pemfile',
            // Contraseña del archivo local_cert.
            'passphrase'        => 'tu_frase_secreta_pem',
            // Permite certificados autofirmados.
            'allow_self_signed' => true,
            // Se debe validar el certificado SSL.
            'verify_peer'       => false
        )
    );

    // Establecer la conexión asíncrona
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Establecer el acceso con encriptación SSL
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
