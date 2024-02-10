# Come inviare e ricevere dati esadecimali

**Ricevere dati esadecimali**

Dopo aver ricevuto i dati, è possibile convertirli in esadecimale utilizzando la funzione ```bin2hex($data)```.

**Inviare dati esadecimali**

Prima di inviare i dati, utilizzare ```hex2bin($data)``` per convertire i dati esadecimali in binario prima di inviarli.

**Esempio:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Ottenere i dati esadecimali
    $hex_data = bin2hex($data);
    // Inviare i dati esadecimali al client
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
