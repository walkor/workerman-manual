# Protocollo WebSocket

Attualmente, la versione del protocollo WebSocket in Workerman è la **13**.

Il protocollo WebSocket è un nuovo protocollo in HTML5. Implementa una comunicazione full-duplex tra il browser e il server.

## Relazione tra WebSocket e TCP

Il WebSocket, come l'HTTP, è un protocollo di livello applicativo basato sulla trasmissione TCP. Il WebSocket non ha molto a che fare con il Socket e non può essere considerato la stessa cosa.

## Handshake del protocollo WebSocket

Il protocollo WebSocket prevede un processo di handshake, durante il quale il browser e il server comunicano tramite il protocollo HTTP. In Workerman, è possibile intervenire in questo processo di handshake in questo modo.

**Per workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // È possibile verificare qui se la connessione proviene da una fonte legittima e chiuderla in caso contrario
        // $_SERVER['HTTP_ORIGIN'] indica da quale sito è stata avviata la connessione WebSocket
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // $_GET e $_SERVER sono accessibili all'interno di onWebSocketConnect
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Per workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## Trasmissione di dati binari tramite il protocollo WebSocket

Il protocollo WebSocket, per impostazione predefinita, può trasmettere solo testo UTF-8. Se si desidera trasmettere dati binari, leggere la seguente sezione.

Nel protocollo WebSocket, nell'intestazione del protocollo, viene utilizzato un flag per indicare se vengono trasmessi dati binari o dati di testo in UTF-8. Il browser verifica che il flag corrisponda al tipo di dati trasmessi e, in caso contrario, disconnette la connessione. Pertanto, quando il server invia i dati, è necessario impostare questo flag in base al tipo di dati trasmessi. In Workerman, se si tratta di un testo UTF-8 normale, è necessario impostare (di default è già impostato e di solito non è necessario farlo manualmente)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Se si tratta di dati binari, è necessario impostare
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Nota**: Se $connection->websocketType non è impostato, allora per impostazione predefinita sarà BINARY_TYPE_BLOB (ossia tipo di testo UTF-8). Di solito le applicazioni trasmettono testi UTF-8, ad esempio dati JSON, quindi non è necessario impostare manualmente $connection->websocketType. Solo nel caso di trasmissione di dati binari (ad esempio dati delle immagini, dati protobuffer, ecc.) è necessario impostare questa proprietà su BINARY_TYPE_ARRAYBUFFER.

## Utilizzo di workerman come client WebSocket

Utilizzando la classe [AsyncTcpConnection](../async-tcp-connection.md) insieme al protocollo [ws](about-ws.md), è possibile fare in modo che workerman agisca come client WebSocket per connettersi a un server WebSocket remoto e completare la comunicazione bidirezionale in tempo reale.
