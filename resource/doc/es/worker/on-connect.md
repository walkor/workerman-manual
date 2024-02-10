# onConnect
## Descripción:

```php
callback Worker::$onConnect
```

La función de devolución de llamada que se activa cuando un cliente establece una conexión con Workerman (después de completar el handshake de TCP). La función ```onConnect``` se ejecutará solo una vez por cada conexión.

Nota: El evento onConnect solo indica que el cliente ha completado el handshake de TCP con Workerman. En este punto, el cliente aún no ha enviado ningún dato, por lo que, aparte de usar ```$connection->getRemoteIp()``` para obtener la dirección IP del cliente, no hay otra información disponible para identificar al cliente. Por lo tanto, no es posible determinar quién es el cliente en el evento onConnect. Para identificar al cliente, se debe enviar datos de autenticación, como un token o nombre de usuario y contraseña, y realizar la autenticación en el [callback onMessage](on-message.md).

Dado que UDP es sin conexión, al usar UDP no se activará el callback onConnect, ni se activará el callback onClose.

## Parámetros de la función de devolución de llamada

``` $connection ```

El objeto de la conexión, es decir, la instancia de [TcpConnection](../tcp-connection.md), se utiliza para operar la conexión del cliente, como [enviar datos](../tcp-connection/send.md) o [cerrar la conexión](../tcp-connection/close.md).


## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "nueva conexión desde la IP " . $connection->getRemoteIp() . "\n";
};
// Ejecutar el worker
Worker::runAll();
```

Nota: Aparte de utilizar funciones anónimas como devolución de llamada, también se puede ver [aquí](../faq/callback_methods.md) otros métodos de devolución de llamada.
