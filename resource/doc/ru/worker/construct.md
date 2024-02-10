# Конструктор __construct

## Описание:
```php
Worker::__construct([string $listen , array $context])
```

Инициализирует экземпляр контейнера Worker с возможностью установки некоторых атрибутов и обратных вызовов для выполнения определенной функциональности.

## Параметры
#### **``` $listen ```** (необязательный, если не указан, то не будет прослушивать какие-либо порты)

Если установлен параметр прослушивания ```$listen```, будет выполнено прослушивание сокета.

Формат $listen: <протокол>://<адрес прослушивания>

**<протокол> может быть в следующем формате:**

tcp: например: ```tcp://0.0.0.0:8686```

udp: например: ```udp://0.0.0.0:8686```

unix: например: ```unix:///tmp/my_file``` (требуется Workerman>=3.2.7)

http: например: ```http://0.0.0.0:80```

websocket: например: ```websocket://0.0.0.0:8686```

text: например: ```text://0.0.0.0:8686``` (text - это встроенный в Workerman текстовый протокол, совместимый с telnet, подробности смотрите в приложении Раздел протокола Text)

а также другие настраиваемые протоколы, см. раздел Настройка протокола связи в данной документации

**<адрес прослушивания> может быть в следующем формате:**

Если это unix-сокет, адресом является путь к локальному файлу на диске.

Если не unix-сокет, формат адреса - <локальный IP-адрес>:<порт>

<локальный IP-адрес> может быть ```0.0.0.0```, что означает прослушивание всех сетевых адаптеров на локальном компьютере, включая внутренние IP-адреса, внешние IP-адреса и адрес обратной петли 127.0.0.1

<локальный IP-адрес> если установлен как ```127.0.0.1```, означает прослушивание адреса обратной петли, к которому можно получить доступ только на локальном компьютере, внешние пользователи не могут обратиться к нему

<локальный IP-адрес>, если это внутренний IP-адрес, например, ```192.168.xx.xx```, означает прослушивание только внутреннего IP-адреса, поэтому внешние пользователи не смогут обратиться к нему

Если установленное значение для <локального IP-адреса> не является локальным IP-адресом компьютера, прослушивание не будет выполнено, и будет выдана ошибка "Cannot assign requested address".

**Примечание:** <порт> не может превышать 65535. Если <порт> меньше 1024, для прослушивания требуются права root. Прослушиваемый порт должен быть свободным на локальном компьютере, в противном случае не удастся выполнить прослушивание и будет выдана ошибка "Address already in use".

#### **``` $context ```**

Массив для передачи параметров контекста сокета, см. [Параметры контекста сокета](https://www.php.net/manual/ru/context.socket.php)

## Примеры

Worker в качестве контейнера http, прослушивающего обработку http-запросов

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// Запуск worker
Worker::runAll();
```

Worker в качестве контейнера websocket, прослушивающего обработку websocket-запросов

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Запуск worker
Worker::runAll();
```

Worker в качестве контейнера tcp, прослушивающего обработку tcp-запросов

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Запуск worker
Worker::runAll();
```

Worker в качестве контейнера udp, прослушивающего обработку udp-запросов

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Запуск worker
Worker::runAll();
```

Worker прослушивает unix-доменный сокет (требуется версия Workerman>=3.2.7)

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Запуск worker
Worker::runAll();
```

Worker не осуществляет прослушивание, а используется для обработки отложенных задач

```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Выполнять каждые 2,5 секунды
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Запуск worker
Worker::runAll();
```

**Worker прослушивает порт с настраиваемым протоколом**

Конечная структура каталогов
```
├── Protocols              // Это каталог Protocols, который нужно создать
│   └── MyTextProtocol.php // Это файл настраиваемого протокола, который нужно создать
├── test.php  // Это скрипт, который нужно создать
└── Workerman // Каталог исходного кода Workerman, в котором код не нужно изменять
```

1. Создание каталога Protocols и файла протокола
Protocols/MyTextProtocol.php (см. структуру каталогов выше)

```php
// Пространство имен для настраиваемого протокола пользователя - Protocols
namespace Protocols;
// Простой текстовый протокол, формат протокола: текст+перевод строки
class MyTextProtocol
{
    // Функция разбиения на пакеты, возвращает длину текущего пакета
    public static function input($recv_buffer)
    {
        // Поиск символа перевода строки
        $pos = strpos($recv_buffer, "\n");
        // Если символ перевода строки не найден, значит пакет не завершен, возвращаем 0 и продолжаем ожидание данных
        if($pos === false)
        {
            return 0;
        }
        // Если символ перевода строки найден, возвращаем длину текущего пакета, включая символ перевода строки
        return $pos+1;
    }

    // После получения полного пакета автоматически вызывается декодирование через decode, здесь просто обрезаем символы перевода строки
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Перед отправкой данных клиенту автоматически вызывается кодирование через encode, затем отправляется клиенту, здесь добавляется символ перевода строки
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. Использование протокола MyTextProtocol для прослушивания и обработки запросов
Создание файла test.php согласно указанной структуре каталогов

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### Worker с протоколом MyTextProtocol ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * После получения полных данных (завершающиеся переводом строки), автоматически выполняется MyTextProtocol::decode('полученные данные')
 * Результат передается через $data для обратного вызова onMessage
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Отправка данных клиенту с автоматическим вызовом MyTextProtocol::encode('hello world') для кодирования протокола,
     * затем отправка клиенту
     */
    $connection->send("hello world");
};

// запуск всех worker-процессов
Worker::runAll();
```

3. Тестирование

Откройте терминал, перейдите в каталог, где находится файл test.php, выполните ```php test.php start```
```php
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Откройте терминал и выполните тестовое подключение с помощью telnet (рекомендуется использовать telnet в системе Linux)

Предположим, что тестирование проводится на локальной машине,
в терминале выполните telnet 127.0.0.1 5678
Затем введите hi и нажмите Enter
Вы получите данные "hello world\n"
```php
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
