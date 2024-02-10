# ouvir
```php
void Worker::listen(void)
```
Usado para instanciar a escuta do Worker.

Este método é principalmente usado para criar dinamicamente novas instâncias de Worker após o início do processo do Worker, permitindo que o mesmo processo escute várias portas e suporte vários protocolos. É importante notar que, ao usar este método, apenas a escuta é aumentada no processo atual, não haverá criação dinâmica de novos processos, nem será acionado o método onWorkerStart.

Por exemplo, após iniciar um Worker http, instanciar um Worker websocket, este processo pode ser acessado tanto por meio do protocolo http quanto do protocolo websocket. Como o Worker websocket e o Worker http estão no mesmo processo, eles podem acessar variáveis de memória comuns e compartilhar todas as conexões de soquete. Isso pode gerar o efeito de receber uma solicitação http e, em seguida, operar com um cliente websocket para enviar dados semelhantes.

**Nota:**

Se a versão do PHP for <=7.0, não será possível instanciar o mesmo Worker na mesma porta em vários processos filhos. Por exemplo, se o Processo A criar um Worker para escutar a porta 2016, então o Processo B não poderá mais criar um Worker para escutar a porta 2016, caso contrário, será gerado um erro de "Address already in use". Por exemplo, o código a seguir não funcionará.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 processos
$worker->count = 4;
// Após cada processo ser iniciado, um novo Worker de escuta será adicionado no processo atual
$worker->onWorkerStart = function($worker)
{
    /**
     * Os 4 processos iniciados criarão um Worker escutando na porta 2016
     * Ao executar worker->listen(), será gerado um erro de "Address already in use"
     * Se worker->count=1, não gerará erro
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Escutar. Neste ponto, será gerado um erro de "Address already in use"
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Executar worker
Worker::runAll();
```

Se a sua versão do PHP for >=7.0, você pode configurar Worker->reusePort=true, o que permite que vários processos filhos criem o mesmo Worker na mesma porta. Veja o exemplo abaixo:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 processos
$worker->count = 4;
// Após cada processo ser iniciado, um novo Worker de escuta será adicionado no processo atual
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Setar porta de reuso, é possível criar um Worker escutando a mesma porta (requer PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Escutar. A escuta normal não gerará erro.
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Executar worker
Worker::runAll();
```


### Exemplo de envio de mensagens em tempo real do back-end PHP para o cliente

**Princípio:**

1. Estabelecer um Worker websocket, para manter conexões longas com o cliente
2. Internamente, o Worker websocket cria um Worker de texto
3. O Worker websocket e o Worker de texto estão no mesmo processo, o que permite compartilhar facilmente conexões com o cliente
4. Um sistema de back-end PHP independente se comunica com o Worker de texto usando o protocolo de texto
5. O Worker de texto opera a conexão do websocket para enviar dados

**Código e Passos:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inicializar um contêiner de worker, ouvindo a porta 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * Observe que o número de processos deve ser configurado como 1
 */
$worker->count = 1;
// Após a inicialização do processo do worker, criar um Worker de texto para abrir uma porta de comunicação interna
$worker->onWorkerStart = function($worker)
{
    // Abrir uma porta interna para facilitar o envio de dados pelo sistema interno. O protocolo de texto é texto+quebra de linha
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // O array $data contém o uid, indicando para qual página enviar os dados
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Por meio do workerman, enviar dados para a página com o uid
        $ret = sendMessageByUid($uid, $buffer);
        // Retornar o resultado do envio
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Escutar ##
    $inner_text_worker->listen();
};
// Adicionar um novo atributo para armazenar a conexão uid
$worker->uidConnections = array();
// Quando um cliente envia uma mensagem
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Verificar se o cliente atual já foi verificado, ou seja, se o uid foi configurado
    if(!isset($connection->uid))
    {
       // Caso não tenha sido verificado, o primeiro pacote é considerado como uid (aqui, para fins de demonstração, nenhuma verificação real é realizada)
       $connection->uid = $data;
       /* Armazenar o uid e a conexão, permitindo encontrar a conexão pelo uid, e realizar o envio de dados para o uid específico */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// Quando um cliente se desconecta
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Ao desconectar, remover o mapeamento
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Enviar dados para todos os usuários verificados
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Enviar dados para um uid específico
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Executar todos os workers
Worker::runAll();
```

Iniciar o serviço back-end
 ```php push.php start -d```

Código JavaScript para receber as notificações no frontend
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

Código para enviar mensagens pelo back-end
```php
// Estabelecer conexão de socket para a porta interna de notificações
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Dados a serem enviados, incluindo o campo uid, indicando para qual uid enviar
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Enviar dados, observe que a porta 5678 é a porta do protocolo de texto, e o protocolo de texto requer uma quebra de linha ao final dos dados
fwrite($client, json_encode($data)."\n");
// Ler o resultado do envio de notificação
echo fread($client, 8192);
```
