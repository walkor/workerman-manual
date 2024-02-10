# Proceso básico
(Usando un simple servidor de chat webSocket como ejemplo)

#### 1. Crear un directorio del proyecto en cualquier lugar
Por ejemplo: SimpleChat/
Ingrese al directorio y ejecute `composer require workerman/workerman`

#### 2. Incluir `vendor/autoload.php` (generado después de instalar composer)
Cree start.php e incluya `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Seleccionar un protocolo
En este caso, seleccionamos el protocolo de texto (un protocolo personalizado en Workerman, con el formato de texto + salto de línea).

(Actualmente Workerman es compatible con HTTP, webSocket, y el protocolo de texto. Si es necesario utilizar otro protocolo, consulte el capítulo de protocolos para desarrollar su propio protocolo).

#### 4. Escribir un script de inicio de entrada según sea necesario
El siguiente es un archivo de entrada simple para una sala de chat.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Cuando un cliente se conecta, asigna un uid, guarda la conexión y notifica a todos los clientes.
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Asigna un uid a esta conexión.
    $connection->uid = ++$global_uid;
}

// Cuando un cliente envía un mensaje, lo reenvía a todos.
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// Cuando un cliente se desconecta, emite una transmisión a todos los clientes.
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// Crea un Worker con protocolo de texto escuchando en el puerto 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// Inicia solo un proceso para facilitar la transferencia de datos entre clientes.
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();

```

#### 5. Pruebas
El protocolo de texto se puede probar con el comando telnet
```shell
telnet 127.0.0.1 2347
```
