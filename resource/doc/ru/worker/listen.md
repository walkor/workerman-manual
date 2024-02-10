# слушать

```php
void Worker::listen(void)
```
Используется для запуска прослушивания после создания экземпляра Worker.

Этот метод в основном используется для динамического создания нового экземпляра Worker после запуска рабочего процесса, что позволяет одному процессу прослушивать несколько портов и поддерживать несколько протоколов. Следует отметить, что использование этого метода приводит только к добавлению прослушивания в текущем процессе, но не создает новый процесс динамически и не вызывает метод onWorkerStart.

Например, после запуска http Worker можно создать экземпляр websocket Worker, чтобы этот процесс мог обслуживать запросы как по протоколу http, так и по протоколу websocket. Поскольку websocket Worker и http Worker находятся в одном процессе, они могут обмениваться общими переменными в памяти и использовать общие соединения с сокетами. Это позволяет принимать запросы по протоколу http, после чего взаимодействовать с клиентом через websocket для передачи данных.

**Примечание:**

Если версия PHP <=7.0, то нельзя создавать Worker, слушающие один и тот же порт в нескольких дочерних процессах. Например, если процесс A создал Worker, слушающий порт 2016, то процесс B не сможет создать Worker, слушающий порт 2016, иначе возникнет ошибка ```Address already in use```. Например, следующий код не сможет быть выполнен.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 процесса
$worker->count = 4;
// После запуска каждого процесса создаем Worker для прослушивания текущего порта в этом процессе
$worker->onWorkerStart = function($worker)
{
    /**
     * При запуске 4 процессов каждый будет создавать Worker для порта 2016
     * При вызове worker->listen() возникнет ошибка Address already in use
     * Если worker->count=1, то ошибка не возникнет
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Начинаем прослушивание. Здесь возникнет ошибка Address already in use
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Запуск worker
Worker::runAll();
```

Если ваша версия PHP >=7.0, можно установить Worker->reusePort=true, чтобы создавать Worker, слушающие один и тот же порт в нескольких дочерних процессах. Пример ниже:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 процесса
$worker->count = 4;
// После запуска каждого процесса создаем Worker для прослушивания текущего порта в этом процессе
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Установка переиспользования порта, что позволяет создавать прослушивание одного и того же порта в разных процессах (требуется PHP >=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Начинаем прослушивание. В этом случае прослушивание будет выполнено без ошибок
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Запуск worker
Worker::runAll();
```


### Пример: немедленная отправка сообщений от сервера PHP клиенту

**Принцип:**

1. Создать WebSocket Worker для поддержания длительного соединения с клиентом.
2. Внутри WebSocket Worker создать text Worker.
3. WebSocket Worker и text Worker находятся в одном процессе, что облегчает обмен клиентскими соединениями.
4. Некоторая отдельная PHP-система на сервере общается с text Worker через протокол text.
5. text Worker управляет соединением WebSocket для отправки данных.

**Код и шаги:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Инициализация контейнера для рабочих процессов, прослушивающих порт 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * Обратите внимание, что количество процессов должно быть установлено равным 1
 */
$worker->count = 1;
// После запуска рабочего процесса создаем text Worker для установки внутреннего порта общения
$worker->onWorkerStart = function($worker)
{
    // Открытие внутреннего порта для удобства отправки данных внутренними системами, формат протокола Text: текст+символ конца строки
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // Массив данных содержит uid, указывающий, кому отправлять данные
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Через workerman отправляем данные на страницу с uid
        $ret = sendMessageByUid($uid, $buffer);
        // Возвращаем результат отправки
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Начинаем прослушивание ##
    $inner_text_worker->listen();
};
// Добавляем свойство, чтобы сохранить соответствие uid и соединений
$worker->uidConnections = array();
// Когда клиент отправляет сообщение
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Проверяем, прошла ли аутентификация текущего клиента, т.е. установлен ли uid
    if(!isset($connection->uid))
    {
       // Если аутентификация не прошла, то первый пакет используем в качестве uid (для демонстрации, без проведения реальной аутентификации)
       $connection->uid = $data;
       /* Сохраняем соответствие uid и соединений, что позволяет легко найти соединение по uid и отправлять сообщения каждому отдельному uid */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// Когда у клиента закрыто соединение
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // При разрыве соединения удаляем соответствие
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Отправка данных всем аутентифицированным пользователям
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Отправка данных определенному uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Запуск всех рабочих процессов
Worker::runAll();
```

Запуск сервера
 ```php push.php start -d```

JS-код для приема полученного PUSH
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

Код для отправки сообщений с сервера
```php
// Устанавливаем соединение с внутренним портом для PUSH-уведомлений
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Данные для отправки содержат поле uid, указывающее, кому отправлять
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Отправляем данные, обратите внимание, что порт 5678 использует протокол Text, который требует символ конца строки в конце данных
fwrite($client, json_encode($data)."\n");
// Получаем результат отправки
echo fread($client, 8192);
```
