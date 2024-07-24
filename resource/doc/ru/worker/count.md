# count

## Описание:
```php
int Worker::$count
```

Устанавливает, сколько процессов будет запущено в текущем экземпляре Worker. Если не установлено, по умолчанию устанавливается значение 1.

Как установить количество процессов, см. [**здесь**](../faq/processes-count.md) .

Примечание: это свойство должно быть установлено перед запуском ```Worker::runAll();```. Эта функциональность не поддерживается в операционной системе Windows.

## Пример

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Запускаем 8 процессов, одновременно прослушивая порт 8484 и предоставляя услуги по протоколу websocket
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Запуск воркера...\n";
};
// Запуск воркера
Worker::runAll();
```