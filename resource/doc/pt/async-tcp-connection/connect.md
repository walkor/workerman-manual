# Método connect
```php
void AsyncTcpConnection::connect()
```
Realiza uma operação de conexão assíncrona. Este método retorna imediatamente.

Nota: Se for necessário definir um retorno de erro assíncrono, isso deve ser feito antes da execução do método connect, caso contrário, o retorno de erro assíncrono pode não ser disparado. Por exemplo, no trecho abaixo, o retorno de erro onError pode não ser acionado e o evento de falha de conexão assíncrona não pode ser capturado.

```php
$connection = new AsyncTcpConnection('tcp://baidu.com:81');
// onError não está definido no momento da conexão
$connection->connect();
$connection->onError = function($connection, $err_code, $err_msg)
{
    echo "$err_code, $err_msg";
};
```

### Parâmetros
Sem parâmetros

### Valor de Retorno
Sem valor de retorno

### Exemplo de proxy Mysql

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Endereço real do mysql, suponha que esteja na porta 3306 local
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// Proxy escuta na porta 4406 local
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function(TcpConnection $connection)
{
    global $REAL_MYSQL_ADDRESS;
    // Estabelece de forma assíncrona uma conexão com o servidor mysql real
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // Quando a conexão mysql envia dados, eles são encaminhados para a conexão do cliente correspondente
    $connection_to_mysql->onMessage = function(AsyncTcpConnection $connection_to_mysql, $buffer) use ($connection)
    {
        $connection->send($buffer);
    };
    // Quando a conexão mysql é fechada, fecha a conexão de proxy correspondente para o cliente
    $connection_to_mysql->onClose = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Se ocorrer um erro na conexão mysql, fecha a conexão de proxy correspondente para o cliente
    $connection_to_mysql->onError = function(AsyncTcpConnection $connection_to_mysql) use ($connection)
    {
        $connection->close();
    };
    // Executa a conexão assíncrona
    $connection_to_mysql->connect();

    // Quando o cliente envia dados, eles são encaminhados para a conexão mysql correspondente
    $connection->onMessage = function(TcpConnection $connection, $buffer) use ($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // Quando a conexão do cliente é fechada, a conexão mysql correspondente também é fechada
    $connection->onClose = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // Se ocorrer um erro na conexão do cliente, a conexão mysql correspondente também é fechada
    $connection->onError = function(TcpConnection $connection) use ($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
};
// Executa o worker
Worker::runAll();
```

 **Teste**

```
mysql -uroot -P4406 -h127.0.0.1 -p

Bem-vindo ao monitor do MySQL. Os comandos terminam com ; ou \g.
Seu ID de conexão MySQL é 25004
Versão do servidor: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. Todos os direitos reservados.

Oracle é uma marca comercial registrada da Oracle Corporation e/ou suas
afiliadas. Outros nomes podem ser marcas comerciais de seus respectivos
proprietários.

Digite 'help;' ou '\h' para obter ajuda. Digite '\c' para limpar a declaração de entrada atual.

mysql>
```
