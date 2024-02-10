# Метод connect
```php
void AsyncTcpConnection::connect()
```
Выполняет асинхронное подключение. Этот метод немедленно возвращает управление.

Примечание: если необходимо установить обратный вызов onError для асинхронного подключения, то его следует установить перед выполнением connect, в противном случае обратный вызов onError может не сработать, например, в следующем примере обратный вызов onError может не сработать, и событие неудачного асинхронного подключения не будет перехвачено.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// Обратный вызов onError еще не установлен во время выполнения подключения
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Параметры
Без параметров

### Возвращаемое значение
Без возвращаемого значения

### Пример: Прокси-сервер Mysql

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Реальный адрес MySQL, предположим, что это localhost и порт 3306
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// Прокси-сервер слушает порт 4406 на локальном устройстве
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Асинхронное подключение к реальному серверу MySQL
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // При получении данных от соединения с MySQL отправляем их соответствующему клиентскому соединению
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // При закрытии соединения с MySQL закрываем соответствующее соединение с клиентом
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // При ошибке соединения с MySQL закрываем соответствующее соединение с клиентом
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Выполняем асинхронное подключение
    $connection_to_mysql->connect();

    // При получении данных от клиента отправляем их соответствующему соединению с MySQL
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // При закрытии соединения с клиентом закрываем соответствующее соединение с MySQL
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // При ошибке соединения с клиентом закрываем соединение с MySQL
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// Запускаем worker
Worker::runAll();
```

**Тестирование**

```bash
mysql -uroot -P4406 -h127.0.0.1 -p

Добро пожаловать в монитор MySQL. Команды заканчиваются точкой с запятой или \g.
ID вашего подключения к MySQL: 25004
Версия сервера: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle и/или его аффилированные лица. Все права защищены.

Oracle является зарегистрированным товарным знаком корпорации Oracle и/или ее аффилированных лиц.
Другие имена могут быть товарными знаками соответствующих владельцев.

Введите 'help;' или '\h' для получения справки. Введите '\c' для очистки текущего ввода.

mysql>
```
