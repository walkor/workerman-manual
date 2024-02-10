# Cliente do componente GlobalData
**``` (Requer Workerman versão >=3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Instancia um objeto cliente \GlobalData\Client. Compartilha dados entre processos atribuindo propriedades ao objeto cliente.

### Parâmetros
Endereço do servidor GlobalData, no formato `<endereço IP>:<porta>`, por exemplo `127.0.0.1:2207`.

Se for um cluster de servidores GlobalData, passe um array de endereços, por exemplo `array('10.0.0.10:2207', '10.0.0.0.11:2207')`.

## Explicação
Suporta operações de atribuição, leitura, isset, unset.
Também suporta operações atômicas cas.

## Exemplos
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Servidor GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// Quando o processo é iniciado
$worker->onWorkerStart = function()
{
    // Inicializa um cliente global data global
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// Sempre que o servidor recebe uma mensagem
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Altera o valor de $global->somedata, outros processos compartilharão essa variável $global->somedata
    global $global;
    echo "agora global->somedata=".var_export($global->somedata, true)."\n";
    echo "definir \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Todos os usos (também pode ser usado em ambiente php-fpm)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));
```

## Observação:
O componente GlobalData não pode compartilhar dados do tipo recurso, como conexões MySQL, conexões de soquete, etc.

Se estiver utilizando o cliente GlobalData/Client em um ambiente Workerman, instancie o objeto GlobalData/Client nos callbacks onXXX, como por exemplo no onWorkerStart.

Não é possível operar variáveis compartilhadas desta forma.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Pode ser feito desta forma
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
