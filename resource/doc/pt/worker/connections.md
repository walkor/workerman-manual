# Conexões
## Descrição:
```php
array Worker::$connections
```

O formato é
```php
array(id=>conexão, id=>conexão, ...)
```

Esta propriedade armazena todos os objetos de conexão do **processo atual**. O id é o número de identificação da conexão, consulte o manual para detalhes sobre a propriedade id de [TcpConnection](../tcp-connection/id.md).

```$connections``` é muito útil para enviar transmissões ou obter um objeto de conexão com base no id da conexão.

Se você souber o id da conexão é ```$id```, pode obter facilmente o objeto de conexão correspondente através de ```$worker->connections[$id]```, e assim operar a conexão do socket correspondente, como enviar dados usando ```$worker->connections[$id]->send('...')```.

Nota: Se a conexão for fechada (desencadear onClose), a correspondente ```connection``` será removida do array ```$connections```.

Nota: Os desenvolvedores não devem modificar esta propriedade, caso contrário, poderá ocorrer situações imprevisíveis.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// Ao iniciar o processo, defina um temporizador para enviar dados para todas as conexões dos clientes periodicamente
$worker->onWorkerStart = function($worker)
{
    // Temporizador, a cada 10 segundos
    Timer::add(10, function()use($worker)
    {
        // Percorre todas as conexões dos clientes do processo atual e envia a hora atual do servidor
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Execute o worker
Worker::runAll();
```
