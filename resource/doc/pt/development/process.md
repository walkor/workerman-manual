# Fluxo Básico
(Usando um servidor de chat Websocket simples como exemplo)

#### 1. Criar o diretório do projeto em qualquer lugar
Por exemplo: SimpleChat/
Entre no diretório e execute `composer require workerman/workerman`

#### 2. Incluir `vendor/autoload.php` (gerado após a instalação do composer)
Crie um arquivo start.php e inclua `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Escolher um protocolo
Aqui nós escolhemos o protocolo de texto (um protocolo personalizado no Workerman, com formato de texto + quebra de linha)

(Atualmente, o Workerman suporta os protocolos HTTP, Websocket e de texto. Se precisar usar outros protocolos, consulte o capítulo de protocolos para desenvolver seu próprio protocolo)

#### 4. Escrever um script de inicialização conforme necessário
Por exemplo, o seguinte é um arquivo de entrada simples para uma sala de chat.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Quando um cliente se conecta, atribui um uid, salva a conexão e notifica todos os clientes
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Atribui um uid para esta conexão
    $connection->uid = ++$global_uid;
}

// Quando um cliente envia uma mensagem, encaminha para todos
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] disse: $data");
    }
}

// Quando um cliente se desconecta, avisa todos os clientes
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] desconectou");
    }
}

// Cria um Worker de protocolo de texto escutando na porta 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// Inicia apenas um processo para facilitar a transmissão de dados entre os clientes
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. Teste
O protocolo de texto pode ser testado usando o comando telnet
```shell
telnet 127.0.0.1 2347
```
