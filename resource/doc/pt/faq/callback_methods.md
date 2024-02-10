# Diversas formas de callback em PHP

Em PHP, a forma mais conveniente de escrever callbacks é usando funções anônimas, mas além disso, existem outras formas de escrever callbacks em PHP. Abaixo estão alguns exemplos das diversas formas de callback em PHP.

## 1. Callback de função anônima
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Callback de função anônima
$http_worker->onMessage = function(TcpConnection $connection, Request $data) {
    // Envia "hello world" para o navegador
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Callback de função comum
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Callback de função comum
$http_worker->onMessage = 'on_message';

// Função comum
function on_message(TcpConnection $connection, Request $request) {
    // Envia "hello world" para o navegador
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Método de classe como callback
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;

class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Script de inicialização start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Carregar MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Criar um objeto
$my_object = new MyClass();

// Chamar os métodos da classe
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Observação: A estrutura de código acima não permite a inicialização de recursos (conexão MySQL, conexão Redis, conexão Memcache, etc.) no construtor, pois o ```$my_object = new MyClass();``` é executado no processo principal. Como exemplo, ao inicializar uma conexão MySQL no processo principal, os subprocessos herdarão esses recursos, cada subprocesso poderá operar essa conexão de banco de dados, no entanto, essas conexões serão as mesmas no servidor MySQL, o que pode levar a erros imprevisíveis, como o erro "mysql gone away".

Se for necessário inicializar recursos no construtor da classe, pode-se usar a seguinte forma.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;

class MyClass {
    protected $db = null;
    public function __construct() {
        // Suponha que a classe de conexão com o banco de dados seja MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Script de inicialização start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Inicializar a classe em onWorkerStart
$worker->onWorkerStart = function($worker) {
    // Carregar MyClass
    require_once __DIR__.'/MyClass.php';
    
    // Criar um objeto
    $my_object = new MyClass();

    // Chamar os métodos da classe
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

No código acima, o onWorkerStart é executado como parte de um subprocesso, o que significa que cada subprocesso cria sua própria conexão MySQL, então não há problema de compartilhamento de conexões. Além disso, isso também suporta a recarga do código de negócios. Como o MyClass.php é carregado no subprocesso, as alterações de código de negócios em MyClass.php terão efeito imediato após a recarga.

## 4. Método estático da classe como callback
MyClass.php estático
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;

class MyClass {
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```
Script de inicialização start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Carregar MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Chamar métodos estáticos da classe.
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Se a classe tiver um namespace, o formato seria semelhante a isso
// $worker->onWorkerStart = array('seu\nomeespaco\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('seu\nomeespaco\MyClass', 'onConnect');
// $worker->onMessage     = array('seu\nomeespaco\MyClass', 'onMessage');
// $worker->onClose       = array('seu\nomeespaco\MyClass', 'onClose');
// $worker->onWorkerStop  = array('seu\nomeespaco\MyClass', 'onWorkerStop');

Worker::runAll();
```

Observação: De acordo com o mecanismo de execução do PHP, 
se o operador `new` não for usado, o construtor não será chamado. Além disso, em métodos estáticos de classe, não é permitido usar o ```$this```.
