# Пример 1
**``` (Требуется Workerman версии >=3.3.0) ```**

Система многопроцессорной отправки на основе рабочего

``` php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Глобальное отображение группы на соединение
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Клиент Channel подключается к серверу Channel
        Channel\Client::connect('127.0.0.1', 2206);

        // Слушаем событие отправки сообщения глобальной группе
        Channel\Client::on('send_to_group', function($event_data){
            $group_id = $event_data['group_id'];
            $message = $event_data['message'];
            global $group_con_map;
            var_dump(array_keys($group_con_map));
            if (isset($group_con_map[$group_id])) {
                foreach ($group_con_map[$group_id] as $con) {
                    $con->send($message);
                }
            }
        });
    };
    $worker->onMessage = function(TcpConnection $con, $data){
        // Присоединиться к сообщению группы {"cmd":"add_group", "group_id":"123"}
        // Или отправить группу сообщение {"cmd":"send_to_group", "group_id":"123", "message":"Это сообщение"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Подключение к группе
            case "add_group":
                global $group_con_map;
                // Добавляем соединение в соответствующий массив группы
                $group_con_map[$group_id][$con->id] = $con;
                // Записываем, к каким группам присоединилось соединение, для удобной очистки данных group_con_map в onclose
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Отправить сообщение группе
            case "send_to_group":
                // Клиент Channel разослать сообщение о событии отправки группы всем серверам и процессам
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // Здесь очень важно удалять соединение из глобальных данных группы при закрытии, чтобы избежать утечки памяти
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Проходим по всем группам, к которым присоединилось соединение, и удаляем соответствующие данные из group_con_map
        if (isset($con->group_id)) {
            foreach ($con->group_id as $group_id) {
                unset($group_con_map[$group_id][$con->id]);
                if (empty($group_con_map[$group_id])) {
                    unset($group_con_map[$group_id]);
                }
            }
        }
    };

    Worker::runAll();
```

## Тестирование (предполагаем, что все работает на локальной машине 127.0.0.1)
1. Запустите сервер
``` 
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          PHP version:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2. Подключение клиента к серверу

Откройте браузер Chrome, нажмите F12, чтобы открыть консоль отладки. Введите следующее в разделе "Console" (или поместите следующий код на HTML-страницу и запустите его с помощью JavaScript):

```javascript
// Предполагается, что IP-адрес сервера 127.0.0.1, измените его на реальный IP-адрес сервера для тестирования
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"Это сообщение"}');
};

```
