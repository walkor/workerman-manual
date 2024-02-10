# Fechar conexão não autenticada

**Pergunta:**

Como fechar a conexão de um cliente que não enviou dados dentro de um intervalo de tempo específico, por exemplo, fechar a conexão se não receber nenhum dado dentro de 30 segundos, para garantir que as conexões não autenticadas precisam ser autenticadas dentro de um tempo especificado.

**Resposta:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // Adiciona temporariamente a propriedade auth_timer_id ao objeto $connection para armazenar o ID do temporizador
    // Fecha a conexão após 30 segundos, se o cliente não enviar a autenticação dentro desse tempo
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ...略
        // Autenticação bem-sucedida, remove o temporizador para evitar o fechamento da conexão
        Timer::del($connection->auth_timer_id);
        break;
         ... 略
    }
    ... 略
}
```
