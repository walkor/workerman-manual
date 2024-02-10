# Esempio di sviluppo semplice

## Installazione

**Installare workerman**
Eseguire il seguente comando in una directory vuota
`composer require workerman/workerman`

## Esempio 1: Fornire servizi Web esterni utilizzando il protocollo HTTP
**Creare il file start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Creare un Worker in ascolto sulla porta 2345, utilizzando il protocollo http per la comunicazione
$http_worker = new Worker("http://0.0.0.0:2345");

// Avviare 4 processi per fornire servizi esterni
$http_worker->count = 4;

// Quando riceve dati inviati dal browser, risponderà con "hello world" al browser
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Inviare "hello world" al browser
    $connection->send('hello world');
};

// Eseguire il Worker
Worker::runAll();
```

**Eseguire da riga di comando (utenti Windows utilizzino [cmd](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn))**
```shell
php start.php start
```

**Test**

Assumendo che l'indirizzo IP del server sia 127.0.0.1

Nel browser, visitare l'URL http://127.0.0.1:2345

 **Nota:**

1. Se si verifica l'inesperienza di accesso, consultare la sezione [Motivi per l'errore di connessione del client](../faq/client-connect-fail.md).

2. Il server utilizza il protocollo http, pertanto può comunicare solo tramite tale protocollo: non è possibile comunicare direttamente tramite websocket o altri protocolli.

## Esempio 2: Fornire servizi esterni utilizzando il protocollo WebSocket
**Creare il file ws_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Nota: qui, a differenza dell'esempio precedente, viene utilizzato il protocollo websocket
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Avviare 4 processi per fornire servizi esterni
$ws_worker->count = 4;

// Quando riceve dati inviati dal client, restituisce "hello $data" al client
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Inviare "hello $data" al client
    $connection->send('hello ' . $data);
};

// Eseguire il Worker
Worker::runAll();
```

**Eseguire da riga di comando**
```shell
php ws_test.php start
```

**Test**

Aprire il browser Chrome, premere F12 per aprire la console di debug, quindi nella console inserire (oppure inserire il seguente codice in una pagina html e eseguirlo tramite javascript)

```javascript
// Assumendo che l'indirizzo IP del server sia 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Connessione riuscita");
    ws.send('tom');
    alert("Inviato al server una stringa: tom");
};
ws.onmessage = function(e) {
    alert("Messaggio ricevuto dal server: " + e.data);
};
```

  **Nota:**

1. Se si verifica l'inesperienza di accesso, consultare la sezione [Motivi per l'errore di connessione del client](../faq/client-connect-fail.md).

2. Il server utilizza il protocollo websocket, pertanto può comunicare solo tramite tale protocollo: non è possibile comunicare direttamente tramite http o altri protocolli.

## Esempio 3: Trasferire direttamente dati tramite TCP
**Creare il file tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Creare un Worker in ascolto sulla porta 2347, senza utilizzare alcun protocollo di livello applicativo
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Avviare 4 processi per fornire servizi esterni
$tcp_worker->count = 4;

// Quando arriva un dato inviato dal client
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Inviare "hello $data" al client
    $connection->send('hello ' . $data);
};

// Eseguire il Worker
Worker::runAll();
```

**Eseguire da riga di comando**

```shell
php tcp_test.php start
```

**Test: Eseguire da riga di comando**
(L'effetto nel terminale di Linux è il seguente, potrebbe variare in ambiente Windows)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**Nota:**

1. Se si verifica l'inesperienza di accesso, consultare la sezione [Motivi per l'errore di connessione del client](../faq/client-connect-fail.md).

2. Il server utilizza il protocollo TCP puro: non è possibile comunicare direttamente tramite websocket, http o altri protocolli.
