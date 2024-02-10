# Объяснение
С 4.x версии workerman значительно усилил поддержку HTTP-сервиса. Был введен класс запроса, класс ответа, класс сеанса и [SSE](SSE.md). Если вы хотите использовать HTTP-сервис workerman, настоятельно рекомендуется использовать версию workerman 4.x или более позднюю.

**Обратите внимание, что все нижеперечисленное относится к использованию версии workerman 4.x и несовместимо с версией workerman 3.x.**

## Изменение механизма хранения сеансов
Workerman предоставляет движки хранения сеансов на основе файлов и Redis. По умолчанию используется двигатель хранения файлов. Чтобы изменить на двигатель хранения Redis, ознакомьтесь с приведенным ниже кодом.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// конфигурация Redis
$config = [
    'host'     => '127.0.0.1', // обязательный параметр
    'port'     => 6379,        // обязательный параметр
    'timeout'  => 2,           // необязательный параметр
    'auth'     => '******',    // необязательный параметр
    'database' => 1,           // необязательный параметр
    'prefix'   => 'session_'   // необязательный параметр
];
// Используйте метод Workerman\Protocols\Http\Session::handlerClass для изменения класса двигателя сеансов
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Установка места хранения сеансов
При использовании двигателя хранения по умолчанию данные сеанса по умолчанию сохраняются на диске в расположении, возвращаемом функцией `session_save_path()`. Вы можете изменить место хранения с помощью следующего метода.

```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Установить место хранения файлов сеанса
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Запустить worker
Worker::runAll();
```

## Очистка файлов сеансов
При использовании двигателя хранения сеансов по умолчанию на диске создается несколько файлов сеансов. Workerman будет очищать устаревшие файлы сеансов в соответствии с параметрами `session.gc_probability`, `session.gc_divisor` и `session.gc_maxlifetime`, установленными в php.ini. См. [документацию по php](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability) для более подробной информации об этих параметрах.

## Изменение хранилища
Помимо файлового и Redis-движков хранения сеансов, workerman позволяет добавлять новые движки хранения сеансов, используя стандартный интерфейс [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php), такие как движок хранения сеансов на основе mangoDB или MySQL.

**Процесс добавления нового двигателя хранения сеансов**
  1.  Реализуйте интерфейс [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php)
  2. Используйте метод `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` для замены базового класса SessionHandlerInterface.

**Реализация интерфейса SessionHandlerInterface**

Пользовательский двигатель хранения сеансов должен реализовывать интерфейс SessionHandlerInterface. Этот интерфейс включает следующие методы:
```php
SessionHandlerInterface {
    /* Методы */
    public function read ( string $session_id ) : string
    public function write ( string $session_id , string $session_data ) : bool
    public function destroy ( string $session_id ) : bool
    public function gc ( int $maxlifetime ) : int
    public function close ( void ) : bool
    public function open ( string $save_path , string $session_name ) : bool
}
```
**Объяснение интерфейса SessionHandlerInterface**
 - Метод read используется для чтения всех данных сеанса, соответствующих идентификатору сеанса, из хранилища. Не выполняйте десериализацию данных, это будет сделано автоматически фреймворком.
 - Метод write используется для записи данных сеанса, соответствующих идентификатору сеанса, в хранилище. Не выполняйте сериализацию данных, это уже сделано автоматически фреймворком.
 - Метод destroy используется для удаления данных сеанса, соответствующих идентификатору сеанса.
 - Метод gc используется для удаления устаревших данных сеанса. Хранилище должно удалять все сеансы, которые не были изменены за время, превышающее maxlifetime.
 - Метод close не требует выполнения каких-либо действий, просто возвращает true.
 - Метод open не требует выполнения каких-либо действий, просто возвращает true.

**Замена базового двигателя**

После реализации интерфейса SessionHandlerInterface используйте следующий метод для замены базового двигателя сеансов.

```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name - имя класса, реализующего интерфейс SessionHandlerInterface. Если класс находится в пространстве имен, укажите полное пространство имен.
 - $config - параметры конструктора класса SessionHandler.

**Конкретная реализация**

*Обратите внимание, что класс MySessionHandler приведен лишь для пояснения процесса замены базового двигателя сеансов, и использовать его в производственной среде не рекомендуется.*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

// Предположим, что новый двигатель SessionHandler требует некоторые параметры для передачи в конструктор
$config = ['host' => 'localhost'];
//  Используйте метод Workerman\Protocols\Http\Session::handlerClass($class_name, $config) для изменения класса базового двигателя сеансов
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
