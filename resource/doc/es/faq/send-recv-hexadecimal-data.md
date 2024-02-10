**Recepción y envío de datos en hexadecimal**

**Recepción de datos en hexadecimal**

Después de recibir los datos, puedes usar la función ```bin2hex($data)``` para convertir los datos a formato hexadecimal.

**Envío de datos en hexadecimal**

Antes de enviar los datos, puedes usar ```hex2bin($data)``` para convertir los datos en formato hexadecimal a binario antes de enviarlos.

**Ejemplo:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Obtener los datos en formato hexadecimal
    $hex_data = bin2hex($data);
    // Enviar los datos en formato hexadecimal al cliente
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
