**Как отправлять и получать данные в шестнадцатеричном формате**

**Получение данных в шестнадцатеричном формате**

После получения данных функцией `bin2hex($data)` можно преобразовать данные в шестнадцатеричный формат.

**Отправка данных в шестнадцатеричном формате**

Перед отправкой данных следует использовать `hex2bin($data)` для преобразования шестнадцатеричных данных в двоичный формат перед отправкой.

**Пример:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // Получение данных в шестнадцатеричном формате
    $hex_data = bin2hex($data);
    // Отправка данных в шестнадцатеричном формате клиенту
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
