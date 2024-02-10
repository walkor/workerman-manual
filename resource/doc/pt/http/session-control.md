# Explicação
A partir da versão 4.x, o Workerman reforçou o suporte para serviços HTTP. Introduziu classes de requisição, de resposta, de sessão e também o [SSE](SSE.md). Se você quiser usar o serviço HTTP do Workerman, é altamente recomendável usar a versão 4.x ou superior.

**Importante: Todas as orientações a seguir referem-se ao uso da versão 4.x do Workerman e não são compatíveis com a versão 3.x.**

## Alterar o mecanismo de armazenamento da sessão
O Workerman disponibiliza o mecanismo de armazenamento de sessão em arquivo e em Redis. O padrão é o armazenamento em arquivo. Se desejar alterar para o armazenamento em Redis, por favor, consulte o código abaixo.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// Configuração do Redis
$config = [
    'host'     => '127.0.0.1', // parâmetro obrigatório
    'port'     => 6379,        // parâmetro obrigatório
    'timeout'  => 2,           // parâmetro opcional
    'auth'     => '******',    // parâmetro opcional
    'database' => 1,           // parâmetro opcional
    'prefix'   => 'session_'   // parâmetro opcional
];
// Use o método Workerman\Protocols\Http\Session::handlerClass para alterar a classe do driver de sessão
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Definir o local de armazenamento da sessão
Ao utilizar o armazenamento padrão, os dados da sessão são armazenados por padrão no disco, na localização retornada por `session_save_path()`. Você pode alterar o local de armazenamento com o seguinte método.
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Configuração do local de armazenamento do arquivo da sessão
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Executar o worker
Worker::runAll();
```

## Limpeza de arquivos de sessão
Ao utilizar o mecanismo de armazenamento padrão da sessão, vários arquivos de sessão podem ser armazenados no disco. O Workerman irá limpar os arquivos de sessão expirados de acordo com as configurações `session.gc_probability`, `session.gc_divisor` e `session.gc_maxlifetime` presentes no php.ini. Para mais informações sobre essas três configurações, consulte o [manual do PHP](https://www.php.net/manual/pt_BR/session.configuration.php#ini.session.gc-probability).

## Alterar o driver de armazenamento
Além dos mecanismos de armazenamento de sessão em arquivo e em Redis, o Workerman permite adicionar novos mecanismos de armazenamento de sessão, como o mecanismo de armazenamento de sessão em MongoDB e em MySQL, através da interface padrão [SessionHandlerInterface](https://www.php.net/manual/pt_BR/class.sessionhandlerinterface.php).

**Processo para adicionar um novo mecanismo de armazenamento de sessão**
1. Implementar a interface [SessionHandlerInterface](https://www.php.net/manual/pt_BR/class.sessionhandlerinterface.php).
2. Utilizar o método `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` para substituir a interface subjacente do SessionHandler.

**Implementar a interface SessionHandlerInterface**

O mecanismo de armazenamento personalizado da sessão deve implementar a interface SessionHandlerInterface, que inclui os seguintes métodos:
```php
SessionHandlerInterface {
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**Explicação da interface SessionHandlerInterface**
- O método read é usado para ler todos os dados da sessão correspondentes ao session_id do armazenamento. Não é necessário deserializar os dados, o framework fará isso automaticamente.
- O método write é usado para escrever os dados da sessão correspondentes ao session_id no armazenamento. Não é necessário serializar os dados, o framework já o faz automaticamente.
- O método destroy é usado para destruir os dados da sessão correspondentes ao session_id.
- O método gc é usado para excluir os dados da sessão expirados, o armazenamento deve executar a exclusão de todas as sessões com horário de última modificação superior a maxlifetime.
- O método close não requer nenhuma operação, apenas retorne true.
- O método open não requer nenhuma operação, apenas retorne true.

**Substituir o driver subjacente**

Após a implementação da interface SessionHandlerInterface, utilize o método a seguir para alterar o driver subjacente da sessão.
```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
- $class_name é o nome da classe SessionHandler que implementa a interface SessionHandlerInterface. Se estiver em um namespace, o namespace completo deve ser incluído.
- $config são os parâmetros do construtor da classe SessionHandler.

**Implementação específica**

*Observação: Esta classe MySessionHandler é apenas para ilustrar o processo de alteração do driver subjacente da sessão e não deve ser utilizada em um ambiente de produção.*
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

// Supondo que a nova classe MySessionHandler implementada requer algum parâmetro de configuração
$config = ['host' => 'localhost'];
// Use o método Workerman\Protocols\Http\Session::handlerClass($class_name, $config) para alterar a classe do driver subjacente da sessão
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
