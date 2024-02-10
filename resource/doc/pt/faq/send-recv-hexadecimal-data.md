# Como enviar e receber dados em hexadecimal

**Receber dados em hexadecimal**

Depois de receber os dados, você pode usar a função ```bin2hex($data)``` para convertê-los em hexadecimal.

**Enviar dados em hexadecimal**

Antes de enviar os dados, use ```hex2bin($data)``` para converter os dados em hexadecimal de volta para binário antes de enviar.

**Exemplo:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Obtendo dados em hexadecimal
    $hex_data = bin2hex($data);
    // Enviando dados em hexadecimal para o cliente
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
