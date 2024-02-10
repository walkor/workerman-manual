# Как реализовать асинхронную задачу

**Вопрос:**

Как обработать тяжелую задачу асинхронно, избегая блокировки основного бизнес-процесса? Например, мне нужно отправить электронное письмо 1000 пользователям, этот процесс занимает много времени, возможно, блокирует работу на несколько секунд, и такие блокировки могут повлиять на последующие запросы. Как передать такую тяжелую задачу для асинхронной обработки другим процессам?

**Ответ:**

Можно заранее создать несколько процессов задач для обработки тяжелых задач на локальной машине или другом сервере, даже на серверном кластере. Количество процессов задач может быть увеличено, например, в 10 раз относительно процессоров, а затем вызывающая сторона может использовать AsyncTcpConnection для асинхронной передачи данных на обработку этим процессам задач и получения результатов асинхронно.

Сервер процессов задач:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Рабочий процесс задач, используя текстовый протокол
$task_worker = new Worker('Text://0.0.0.0:12345');
// Количество рабочих процессов задач может быть увеличено по необходимости
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // Предположим, что пришли данные в формате JSON
     $task_data = json_decode($task_data, true);
     // Обработка задачи на основе task_data.... Получение результата; здесь опущено....
     $task_result = ......
     // Отправка результата
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Вызов из Workerman:

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Websocket-сервер
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Установить асинхронное соединение с удаленным сервисом задач, ip-адресом которого является ip-адрес удаленного сервиса задач; если это локальная машина, то 127.0.0.1, если кластер, тогда ip-адрес LVS
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Данные задачи и параметры
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Отправка данных
    $task_connection->send(json_encode($task_data));
    // Асинхронное получение результата
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // Результат
         var_dump($task_result);
         // После получения результата не забудьте закрыть асинхронное соединение
         $task_connection->close();
         // Уведомление соответствующего веб-сокет-клиента о завершении задачи
         $ws_connection->send('задача завершена');
    };
    // Выполнить асинхронное соединение
    $task_connection->connect();
};

Worker::runAll();
```

Таким образом, тяжелые задачи передаются для выполнения на локальной машине или другом сервере, и после завершения задачи результат получается асинхронно, основной бизнес-процесс больше не блокируется.
