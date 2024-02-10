# Comment envoyer et recevoir des données en hexadécimal

**Recevoir des données en hexadécimal**

Après avoir reçu des données, utilisez la fonction `bin2hex($data)` pour convertir les données en hexadécimal.

**Envoyer des données en hexadécimal**

Avant d'envoyer des données, utilisez `hex2bin($data)` pour convertir les données hexadécimales en binaire avant de les envoyer.

**Exemple :**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Obtenir des données en hexadécimal
    $hex_data = bin2hex($data);
    // Envoyer des données en hexadécimal au client
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
