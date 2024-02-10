# Constructor __construct

## Descripción
```php
Worker::__construct([string $listen , array $context])
```

Inicializa una instancia del contenedor Worker, y se pueden configurar algunas propiedades y callbacks de la instancia para lograr funciones específicas.

## Parámetros
#### **``` $listen ```** (Opcional, no se indica para no escuchar ningún puerto)

Si se hace la configuración de parámetros de escucha ```$listen```, entonces se lleva a cabo la escucha del socket.

El formato de ```$listen``` es <protocolo>://<dirección de escucha>

**<protocolo> puede ser en los siguientes formatos:** 

tcp: por ejemplo ```tcp://0.0.0.0:8686```

udp: por ejemplo ```udp://0.0.0.0:8686```

unix: por ejemplo ```unix:///tmp/my_file``` ```(requiere Workerman>=3.2.7)```

http: por ejemplo ```http://0.0.0.0:80```

websocket: por ejemplo ```websocket://0.0.0.0:8686```

text: por ejemplo ```text://0.0.0.0:8686``` ```(text es un protocolo de texto integrado en Workerman, compatible con telnet, para más detalles ver la sección de Apéndice Text Protocol)```

y otros protocolos personalizados, consulta la parte de este manual sobre la personalización de protocolos de comunicación

**<dirección de escucha> puede estar en los siguientes formatos:**

Si se trata de un socket de dominio unix, la dirección es una ruta de disco local

Para los sockets que no son de dominio unix, el formato de la dirección es <IP de la máquina local>:<número de puerto>

<IP de la máquina local> puede ser```0.0.0.0``` para escuchar en todas las interfaces de red de la máquina local, incluyendo la IP de la red local, la IP de la red externa, así como el localhost 127.0.0.1

Si <IP de la máquina local> comienza con ```127.0.0.1``` entonces escuchará solamente el localhost, únicamente accesible desde la máquina local y no desde el exterior

Si <IP de la máquina local> es una IP de red interna, similar a ```192.168.xx.xx```, entonces solamente escuchará la IP de red interna, por lo que los usuarios de la red externa no podrán acceder

Si el valor de <IP de la máquina local> no coincide con ninguna dirección IP de la máquina local, no será posible llevar a cabo la escucha, y mostrará una advertencia de ```Cannot assign requested address```

**Nota:** <número de puerto> no puede ser mayor a 65535. Si <número de puerto> es menor a 1024, se necesita permisos de administrador para poder escuchar. El puerto de escucha debe ser un puerto desocupado en la máquina local, de lo contrario no se podrá escuchar y mostrará un error de ```Address already in use```

#### **``` $context ```**

Un array para pasar las opciones de contexto del socket, consulta [Opciones de contexto del socket](https://php.net/manual/zh/context.socket.php)

## Ejemplo

Worker como contenedor http para escuchar y procesar peticiones http
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```

Worker como contenedor websocket para escuchar y procesar peticiones websocket
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```

Worker como contenedor tcp para escuchar y procesar peticiones tcp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```

Worker como contenedor udp para escuchar y procesar peticiones udp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```

Worker que escucha los sockets de dominio unix ```(requiere Workerman versión>=3.2.7)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```

El contenedor Worker sin realizar ninguna escucha se utiliza para procesar algunas tareas programadas
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Ejecutar cada 2.5 segundos
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Ejecutar el worker
Worker::runAll();
```

**Contenedor Worker escuchando puerto con protocolo personalizado**

Estructura de directorio final
```
├── Protocols              // Este es el directorio Protocols a crear
│   └── MyTextProtocol.php // Este es el archivo de protocolo personalizado a crear
├── test.php  // Este es el script de prueba a crear
└── Workerman // Directorio de código fuente de Workerman, el código aquí no debe ser modificado
```

1. Crear el directorio Protocols y crear un archivo de protocolo
Protocols/MyTextProtocol.php (según la estructura del directorio mencionada anteriormente)

```php
// El espacio de nombres para los protocolos personalizados es Protocols
namespace Protocols;
// Protocolo de texto simple, formato del protocolo: texto + salto de línea
class MyTextProtocol
{
    // Función de paquete, devuelve la longitud del paquete actual
    public static function input($recv_buffer)
    {
        // Buscar el salto de línea
        $pos = strpos($recv_buffer, "\n");
        // Si no se encuentra el salto de línea, significa que no es un paquete completo, devuelve 0 para seguir esperando datos
        if($pos === false)
        {
            return 0;
        }
        // Encuentra el salto de línea, devuelve la longitud actual del paquete, incluyendo el salto de línea
        return $pos+1;
    }

    // Una vez que se recibe un paquete completo, se ejecuta automáticamente la decodificación a través de decode('datos recibidos') y se pasa el resultado al callback onMessage
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Antes de enviar datos al cliente, se llama automáticamente a encode para codificar el protocolo, y luego se envía al cliente, aquí se agrega un salto de línea
    public static function encode($data)
    {
        return $data."\n";
    }
}
```
2. Utilizando el protocolo MyTextProtocol para escuchar y procesar peticiones

De acuerdo con la estructura del directorio final mencionada anteriormente, se crea el archivo test.php

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### Worker con MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * Una vez que se recibe un paquete completo (con salto de línea), se ejecuta automáticamente MyTextProtocol::decode('datos recibidos')
 * El resultado se pasa a onMessage a través de $data
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Enviar datos al cliente, automáticamente se llama a MyTextProtocol::encode('hello world') para codificar el protocolo,
     * y luego se envía al cliente
     */
    $connection->send("hello world");
};

// Ejecutar todos los workers
Worker::runAll();
```
3. Prueba

Abrir la terminal, ingresar al directorio donde se encuentra test.php y ejecutar ```php test.php start```
```shell
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Abrir la terminal y usar telnet (se recomienda utilizar telnet en un sistema Linux)

Suponiendo que se está probando en la propia máquina,
en la terminal ejecutar telnet 127.0.0.1 5678
luego escribir hi y presionar enter
Recibirás la respuesta hello world\n
```shell
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
