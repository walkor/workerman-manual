# stopAll
```php
void Worker::stopAll(void)
```

Detiene el proceso actual y sale.  

> **Nota**
> `Worker::stopAll()` se utiliza para detener el proceso actual. Una vez que el proceso actual sale, el proceso principal iniciará inmediatamente un nuevo proceso. Si deseas detener todo el servicio de workerman, por favor utiliza `posix_kill(posix_getppid(), SIGINT)`.

### Parámetros
Sin parámetros.

### Valor de retorno
Sin valor de retorno

## Ejemplo max_request

En el siguiente ejemplo, el subproceso ejecuta stopAll para salir después de procesar 1000 peticiones, con el propósito de reiniciar un nuevo proceso completamente nuevo. Es similar a la propiedad max_request de php-fpm y se usa principalmente para resolver problemas de fuga de memoria causados por errores en el código del negocio de php.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Máximo 1000 peticiones por proceso
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Número de peticiones procesadas
    static $request_count = 0;

    $connection->send('hello http');
    // Si el número de peticiones alcanza 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Salir del proceso actual, el proceso principal iniciará inmediatamente
         * un nuevo proceso completamente nuevo para reemplazarlo y así completar
         * el reinicio del proceso.
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
