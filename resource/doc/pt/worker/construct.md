# Construtor __construct

## Descrição:
```php
Worker::__construct([string $listen , array $context])
```

Inicializa uma instância de contêiner Worker, pode configurar algumas propriedades do contêiner e interfaces de retorno de chamada para concluir funções específicas.

## Parâmetros
#### **``` $listen ```** (parâmetro opcional, se não preenchido, significa que não está ouvindo em nenhuma porta)

Se o parâmetro `listen` estiver configurado, ele executará a escuta do soquete.

O formato de `$listen` é <protocolo>://<endereço de escuta>

**<protocolo> pode ser nos seguintes formatos:** 

tcp: Por exemplo, `tcp://0.0.0.0:8686`

udp: Por exemplo, `udp://0.0.0.0:8686`

unix: Por exemplo, `unix:///tmp/my_file` (requer Workerman>=3.2.7)

http: Por exemplo, `http://0.0.0.0:80`

websocket: Por exemplo, `websocket://0.0.0.0:8686`

text: Por exemplo, `text://0.0.0.0:8686` (text é um protocolo de texto incorporado no Workerman, compatível com telnet, consulte a seção do Protocolo de Texto no apêndice para mais detalhes)

e outros protocolos personalizados, consulte a seção de Personalização de Protocolo de Comunicação neste manual

**<endereço de escuta> pode ser nos seguintes formatos:**

Se for um soquete Unix, o endereço é um caminho de arquivo local.

Se não for um soquete Unix, o formato do endereço é <ip local>:<número da porta>.

<ip local> pode ser `0.0.0.0` para escutar em todas as placas de rede locais, incluindo IP internos, IP externos e localhost 127.0.0.1.

Se <ip local> for `127.0.0.1`, ele escuta apenas no localhost, acessível somente localmente e não externamente.

Se <ip local> for um IP interno, como `192.168.xx.xx`, ele escutará apenas no IP interno, impedindo que usuários externos acessem.

Se <ip local> não for um IP local, a escuta não será executada e um erro `Cannot assign requested address` será exibido.

**Observação:** `<número da porta>` não pode ser superior a 65535. Se for inferior a 1024, é necessário permissão de root para escutar. A porta ouvida deve ser uma porta não utilizada na máquina local; caso contrário, não será possível escutar e um erro `Address already in use` será exibido.

#### **``` $context ```**

Um array. Utilizado para passar opções de contexto do soquete, consulte [Opções de Contexto de Soquete](https://php.net/manual/pt_BR/context.socket.php)

## Exemplos

Worker atuando como contêiner http ouvindo e tratando solicitações http
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// Executa todos os workers
Worker::runAll();
```

Worker atuando como contêiner websocket ouvindo e tratando solicitações websocket
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Executa todos os workers
Worker::runAll();
```

Worker atuando como contêiner tcp ouvindo e tratando solicitações tcp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Executa todos os workers
Worker::runAll();
```

Worker atuando como contêiner udp ouvindo e tratando solicitações udp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Executa todos os workers
Worker::runAll();
```

Worker ouvindo soquete de domínio unix ```(requer Workerman versão >= 3.2.7)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Executa todos os workers
Worker::runAll();
```

Contêiner Worker que não executa nenhum tipo de escuta, usado para processar tarefas agendadas
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Executa a cada 2,5 segundos
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Executa todos os workers
Worker::runAll();
```

**Worker ouvindo em uma porta com um protocolo personalizado**

Estrutura final do diretório
```
├── Protocols              // Este é o diretório Protocols a ser criado
│   └── MyTextProtocol.php // Este é o arquivo de protocolo personalizado a ser criado
├── test.php  // Este é o script test a ser criado
└── Workerman // Diretório de código-fonte do Workerman, o código lá dentro não deve ser alterado
```

1. Cria o diretório Protocols e um arquivo de protocolo
Protocols/MyTextProtocol.php (de acordo com a estrutura do diretório acima)

```php
// O namespace dos protocolos personalizados é unificado como Protocols
namespace Protocols;
// Protocolo de texto simples, o formato do protocolo é texto + quebra de linha
class MyTextProtocol
{
    // Função de empacotamento, retorna o comprimento do pacote atual
    public static function input($recv_buffer)
    {
        // Procura o caractere de quebra de linha
        $pos = strpos($recv_buffer, "\n");
        // Se não encontrar o caractere de quebra de linha, indica que não é um pacote completo, retorna 0 para continuar aguardando dados
        if($pos === false)
        {
            return 0;
        }
        // Se encontrar o caractere de quebra de linha, retorna o comprimento do pacote atual, incluindo o caractere de quebra de linha
        return $pos+1;
    }

    // Após receber um pacote completo, ele é automaticamente decodificado usando o método decode. Aqui, apenas remove a quebra de linha.
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Antes de enviar dados ao cliente, eles são automaticamente codificados pelo método encode e, em seguida, enviados ao cliente. Aqui é adicionada uma quebra de linha.
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. Usando o protocolo MyTextProtocol para ouvir e tratar solicitações

Criar o arquivo test.php, de acordo com a estrutura de diretórios acima
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### Worker do MyTextProtocol ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * Após receber os dados completos (terminando com uma quebra de linha), o MyTextProtocol::decode('dados recebidos') é executado automaticamente.
 * O resultado é passado para a função de retorno de chamada onMessage através de $data.
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Envia dados para o cliente. Ao fazer isso, o MyTextProtocol::encode('hello world') é chamado automaticamente para codificar o protocolo,
     * e depois é enviado para o cliente
     */
    $connection->send("hello world");
};

// Executa todos os workers
Worker::runAll();
```

3. Testando

Abra o terminal e acesse o diretório onde está o test.php, execute `php test.php start`
```
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Abra o terminal e teste usando telnet (recomendável usar telnet em um sistema Linux)

Supondo que seja um teste local,
No terminal, digite telnet 127.0.0.1 5678
Em seguida, insira hi e pressione Enter
Você receberá a resposta hello world\n
```
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
