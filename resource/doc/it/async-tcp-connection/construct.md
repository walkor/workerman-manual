# Metodo __construct
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Crea un oggetto di connessione asincrona.

AsyncTcpConnection consente a Workerman di fungere da client per avviare una connessione asincrona con un server remoto e inviare e gestire in modo asincrono i dati sulla connessione tramite l'interfaccia di invio e il callback onMessage.

## Parametri
Parametro: `remote_address`

L'indirizzo di connessione, ad esempio
```
tcp://www.baidu.com:80
ssl://www.baidu.com:443
ws://echo.websocket.org:80
frame://192.168.1.1:8080
text://192.168.1.1:8080
```

Parametro: `context_option`

`Questo parametro richiede (workerman >= 3.3.5)`

Utilizzato per impostare il contesto del socket, ad esempio utilizzando `bindto` per stabilire con quale (scheda di rete) indirizzo IP e porta accedere alla rete esterna, impostando certificati SSL, ecc.

Vedi [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Opzioni del contesto del socket](https://php.net/manual/zh/context.socket.php), [Opzioni del contesto SSL](https://php.net/manual/zh/context.ssl.php)

## Nota

Attualmente AsyncTcpConnection supporta i protocolli [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md).

Supporta anche i protocolli personalizzati, consultare [Come personalizzare i protocolli](../protocols/how-protocols.md)

Il protocollo [ssl](https://baike.baidu.com/view/525499.htm) richiede Workerman>=3.3.4 e l'installazione dell'estensione [openssl](https://php.net/manual/zh/book.openssl.php).

Attualmente AsyncTcpConnection non supporta il protocollo [http](https://baike.baidu.com/view/9472.htm).

È possibile utilizzare `new AsyncTcpConnection('ws://...')` per avviare una connessione WebSocket remota come un browser in Workerman, vedi [Esempio](../appendices/about-ws.md). Ma non è possibile avviare una connessione WebSocket in Workerman utilizzando `new AsyncTcpConnection('websocket://...')`.

## Esempi

### Esempio 1: Accesso asincrono a un servizio http esterno
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Allo start del processo, stabilisce in modo asincrono una connessione a www.baidu.com e invia i dati per ottenere una risposta
$task->onWorkerStart = function($task)
{
    // Non supporta la specifica diretta di http, ma è possibile usare tcp per simulare l'invio del protocollo http
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // Quando la connessione è stabilita con successo, invia i dati della richiesta http
    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connessione stabilita con successo\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Connessione chiusa\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Codice errore: $code messaggio: $msg\n";
    };
    $connection_to_baidu->connect();
};

// Avvia il worker
Worker::runAll();
```

### Esempio 2: Accesso asincrono a un servizio websocket esterno e impostazione dell'ip e della porta locale di accesso
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Imposta l'ip e la porta locale per accedere all'host remoto (ogni connessione socket occupa una porta locale)
    $context_option = array(
        'socket' => array(
            // L'ip deve essere l'ip della scheda di rete locale e deve essere in grado di accedere all'host remoto, altrimenti sarà invalido
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('ciao');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### Esempio 3: Accesso asincrono a una porta wss esterna e configurazione del certificato SSL locale
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Imposta l'ip e la porta locale per accedere all'host remoto, nonché il certificato SSL locale
    $context_option = array(
        'socket' => array(
            // L'ip deve essere l'ip della scheda di rete locale e deve essere in grado di accedere all'host remoto, altrimenti sarà invalido
            'bindto' => '114.215.84.87:2333',
        ),
        // Opzioni SSL, vedere https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Percorso del certificato locale. Deve essere in formato PEM e includere il certificato locale e la chiave privata.
            'local_cert'        => '/percorso/tuo/file.pem',
            // Password del file local_cert.
            'passphrase'        => 'la_tua_passphrase_del_pem',
            // Se consentire certificati auto-firmati.
            'allow_self_signed' => true,
            // Se è necessaria la convalida del certificato SSL.
            'verify_peer'       => false
        )
    );

    // Avvia la connessione asincrona
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Imposta l'accesso tramite SSL
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('ciao');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
