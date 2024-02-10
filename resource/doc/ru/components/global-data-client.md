# Клиент компонента GlobalData
**``` (требуется Workerman версии >=3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Создает объект клиента \GlobalData\Client. Позволяет осуществлять обмен данными между процессами путем установки значений свойств объекта клиента.

### Параметры
Адрес сервера GlobalData в формате ```<IP-адрес>:<порт>```, например, ```127.0.0.1:2207```.

Если сервер GlobalData является кластером, передается массив адресов, например, ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```.

## Описание
Поддерживает операции присваивания, чтения, проверки существования и удаления.
Также поддерживает атомарную операцию cas.

## Пример

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Сервер GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// При запуске процесса
$worker->onWorkerStart = function()
{
    // Инициализация глобального клиента global data
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// При каждом получении сообщения сервером
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Изменение значения $global->somedata, другие процессы будут разделять это значение
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Полный пример использования (можно также использовать в среде php-fpm)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## Важно:
Компонент GlobalData не может обмениваться данными ресурсного типа, например, подключения к MySQL, сокеты и т. д. невозможно разделять.

Если вы используете компонент GlobalData/Client в среде Workerman, следует создавать объект GlobalData/Client в коллбэках onXXX, например, в onWorkerStart.

Нельзя выполнять такие операции с общими переменными.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Можно сделать следующим образом
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
