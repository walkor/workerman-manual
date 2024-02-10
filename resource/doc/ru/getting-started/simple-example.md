# Простой пример разработки

## Установка

**Установка Workerman**

Запустите следующую команду в пустой директории:

`composer require workerman/workerman`

## Пример 1. Предоставление веб-сервиса с использованием протокола HTTP
**Создание файла start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Создание Worker, прослушивающего порт 2345 и использующего протокол HTTP
$http_worker = new Worker("http://0.0.0.0:2345");

// Запуск 4 процессов для предоставления сервиса
$http_worker->count = 4;

// При получении данных от браузера, отправлять "hello world" обратно
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Отправляем "hello world" в браузер
    $connection->send('hello world');
};

// Запуск worker'а
Worker::runAll();
```

**Запуск через командную строку (для пользователей Windows используйте [командную строку cmd](https://ru.wikipedia.org/wiki/Cmd.exe))**
```shell
php start.php start
```

**Тестирование**

Предположим, что IP-адрес сервера - 127.0.0.1

Откройте браузер и введите URL http://127.0.0.1:2345

 **Примечание:**

1. Если возникают проблемы с доступом, пожалуйста, ознакомьтесь с разделом [Причины сбоя подключения клиента](../faq/client-connect-fail.md).

2. Сервер использует протокол HTTP, поэтому можно использовать только HTTP для общения. Другие протоколы, такие как WebSocket, не поддерживаются.

## Пример 2. Предоставление сервиса с использованием протокола WebSocket
**Создание файла ws_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Обратите внимание: в этом примере используется протокол WebSocket
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Запуск 4 процессов для предоставления сервиса
$ws_worker->count = 4;

// При получении данных от клиента, отправить "hello $data" обратно
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Отправляем "hello $data" клиенту
    $connection->send('hello ' . $data);
};

// Запуск worker'а
Worker::runAll();
```

**Запуск через командную строку**
```shell
php ws_test.php start
```

**Тестирование**

Откройте браузер Chrome, нажмите F12 для открытия консоли разработчика, вкладка Console, выполните следующий код (или поместите этот код в HTML-страницу и запустите его с помощью JavaScript):

```javascript
// Предположим, что IP-адрес сервера - 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Соединение установлено");
    ws.send('tom');
    alert("Отправлено серверу: tom");
};
ws.onmessage = function(e) {
    alert("Получено сообщение от сервера: " + e.data);
};
```

  **Примечание:**

1. Если возникают проблемы с доступом, пожалуйста, ознакомьтесь с разделом [Сбой подключения клиента](../faq/client-connect-fail.md) в руководстве.

2. Сервер использует протокол WebSocket, поэтому можно использовать только WebSocket для общения. Другие протоколы, такие как HTTP, не поддерживаются.

## Пример 3. Прямая передача данных по протоколу TCP
**Создание файла tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Создание Worker, прослушивающего порт 2347 без использования прикладного уровня протокола
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Запуск 4 процессов для предоставления сервиса
$tcp_worker->count = 4;

// При получении данных от клиента
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Отправляем "hello $data" клиенту
    $connection->send('hello ' . $data);
};

// Запуск worker'а
Worker::runAll();
```

**Запуск через командную строку**

```shell
php tcp_test.php start
```

**Тестирование через командную строку**
(Приведены команды для Linux; в Windows результат может отличаться)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**Примечание:**

1. Если возникают проблемы с доступом, пожалуйста, ознакомьтесь с разделом [Сбой подключения клиента](../faq/client-connect-fail.md) в руководстве.

2. Сервер использует голый протокол TCP, поэтому нельзя использовать прямое общение с помощью протоколов, таких как WebSocket или HTTP.
