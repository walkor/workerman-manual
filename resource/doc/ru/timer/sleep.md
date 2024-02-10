```php
int \Workerman\Timer::sleep(float $delay)
```
Аналогический функции `sleep()` встроенной в PHP, за исключением того, что `Timer::sleep()` не блокирует процесс.

> **Примечание**
> Данная функция требует workerman>=5.0
> Для использования этой функции необходимо установить composer require revolt/event-loop ^1.0.0 или использовать Swoole/Swow в качестве драйвера событий

### Параметры
``` delay ```

Время до выполнения в секундах, может содержать десятичную часть, точность до 0.001, т.е. до миллисекунд.

### Возвращаемые значения
Отсутствует

### Пример

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // Отправить данные с задержкой в 1.5 секунды
    Timer::sleep(1.5);
    // Отправить данные
    $connection->send('привет, workerman');
};

Worker::runAll();
```
