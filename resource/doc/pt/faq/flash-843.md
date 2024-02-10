# Abrir a porta 843 para o Flash

Quando o Flash inicia a conexão de soquete com o servidor remoto, ele primeiro solicita um arquivo de política de segurança na porta 843 do servidor correspondente. Caso contrário, o Flash não pode estabelecer uma conexão com o servidor. No Workerman, você pode abrir a porta 843 e retornar o arquivo de política de segurança da seguinte forma:

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

O conteúdo da política de segurança em XML pode ser personalizado de acordo com suas necessidades.
