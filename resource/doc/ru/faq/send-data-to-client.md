# Как отправить данные на конкретного клиента в Workerman
Если вы используете worker в качестве сервера, а не GatewayWorker, то как реализовать отправку сообщений определенному пользователю?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Инициализация контейнера worker, слушающего порт 1234
$worker = new Worker('websocket://workerman.net:1234');
// ==== ЗДЕСЬ КОЛИЧЕСТВО ПРОЦЕССОВ ДОЛЖНО БЫТЬ УСТАНОВЛЕНО В 1 ====
$worker->count = 1;
// Добавление нового свойства для сохранения соответствия uid и соединения (uid - идентификатор пользователя или уникальный идентификатор клиента)
$worker->uidConnections = array();
// Функция обратного вызова, когда клиент отправляет сообщение
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Проверяем, прошел ли текущий клиент проверку, т.е. установлен ли uid
    if(!isset($connection->uid))
    {
       // Если не прошел, то первый пакет считается uid (для удобства демонстрации, здесь не выполняется реальная проверка)
       $connection->uid = $data;
       /* Сохраняем соответствие uid и соединения, таким образом, можно легко найти соединение по uid и отправлять данные для конкретного uid */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // Другая логика, отправка данных конкретному uid или глобальное распространение
    // Предположим, что формат сообщения uid:message означает отправку сообщения для uid
    // uid - all означает глобальную рассылку
    list($recv_uid, $message) = explode(':', $data);
    // Глобальная рассылка
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // Отправка конкретному uid
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// При отключении клиента
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Удаление соответствия при отключении соединения
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Отправка данных всем авторизованным пользователям
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Отправка данных по uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// Запуск всех worker (фактически определен только один)
Worker::runAll();
```
**Примечание:**

В приведенном выше примере можно осуществлять отправку по uid, хотя процесс является однопроцессным, но он поддерживает без проблем 10 тыс. онлайн.

Обратите внимание, что в этом примере может быть только один процесс, то есть $worker->count должно быть равно 1. Чтобы поддерживать несколько процессов или кластер серверов, необходимо использовать компонент Channel для межпроцессного взаимодействия. Разработка также очень проста, и вы можете обратиться к разделу [Пример кластеризации компонента Channel](../components/channel-examples.md).

**Если вы хотите отправлять сообщения клиентам из других систем, обратитесь к разделу [Отправка сообщений в другом проекте](push-in-other-project.md)**
