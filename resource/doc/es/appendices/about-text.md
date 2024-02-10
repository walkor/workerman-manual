# Protocolo de texto
Workerman define un protocolo de texto llamado "text", cuyo formato es ```paquete de datos + salto de línea```, es decir, se agrega un salto de línea al final de cada paquete de datos para indicar el final del paquete.

Por ejemplo, las cadenas de buffer1 y buffer2 a continuación cumplen con el protocolo de texto:

```php
// Texto con un salto de línea
$buffer1 = 'abcdefghijklmn
';
// En PHP, \n dentro de comillas dobles representa un salto de línea, por ejemplo, "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Establecer una conexión de socket con el servidor
$client = stream_socket_client('tcp://127.0.0.1:5678');
// Enviar datos buffer1 utilizando el protocolo de texto
fwrite($client, $buffer1);
// Enviar datos buffer2 utilizando el protocolo de texto
fwrite($client, $buffer2);
```

El protocolo de texto es muy simple y fácil de usar. Si los desarrolladores necesitan un protocolo propio, como la transferencia de datos con una aplicación para móviles o la comunicación con hardware, pueden considerar utilizar el protocolo de texto, ya que resulta muy conveniente para el desarrollo y la depuración.

**Depuración del protocolo de texto**

El protocolo de texto se puede depurar utilizando un cliente telnet, como se muestra en el siguiente ejemplo:

Crear el archivo test.php

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage = function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

Ejecutar ```php test.php start``` muestra lo siguiente

```php
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

Abrir una nueva terminal y probar con telnet (se recomienda usar telnet en un sistema Linux)

Suponiendo que es una prueba local,
Ejecutar telnet 127.0.0.1 5678
Luego ingresar hi y presionar enter
Se recibirá la respuesta hello world\n
```php
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
