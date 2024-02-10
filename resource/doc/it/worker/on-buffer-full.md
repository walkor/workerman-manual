# onBufferFull
## Descrizione:
```php
callback Worker::$onBufferFull
```

Ogni connessione ha un buffer di invio a livello di applicazione separato. Se la velocità di ricezione del client è inferiore alla velocità di invio del server, i dati verranno temporaneamente memorizzati nel buffer di invio a livello di applicazione e, se il buffer è pieno, verrà attivata la callback onBufferFull.

Il buffer massimo è [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md), il cui valore predefinito è di 1 MB. È possibile impostare dinamicamente la dimensione del buffer per la connessione corrente, ad esempio:
```php
// Imposta la dimensione del buffer di invio per la connessione corrente, in byte
$connection->maxSendBufferSize = 102400;
```
È anche possibile utilizzare [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) per impostare la dimensione predefinita del buffer di invio per tutte le connessioni, come mostrato nel seguente esempio:
```php
use Workerman\Connection\TcpConnection;
// Imposta la dimensione predefinita del buffer di invio a livello di applicazione per tutte le connessioni, in byte
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

Questa callback **potrebbe** essere attivata immediatamente dopo la chiamata a Connection::send, ad esempio quando vengono inviati grandi dati o quando vengono inviati rapidamente dati al peer. A causa di problemi di rete, i dati vengono accumulati in modo massiccio nel buffer di invio della connessione corrispondente, attivando la callback quando si supera il limite di ```TcpConnection::$maxSendBufferSize```.

Quando si verifica l'evento onBufferFull, di solito è necessario adottare misure, come smettere di inviare dati al peer e attendere il completamento dell'invio dei dati nel buffer (evento onBufferDrain).

Quando si chiama Connection::send(`$A`) e si attiva onBufferFull, indipendentemente dalla dimensione dei dati `$A` inviati questa volta, anche se superano ```TcpConnection::$maxSendBufferSize```, i dati da inviare verranno comunque inseriti nel buffer di invio. In altre parole, i dati effettivamente inseriti nel buffer di invio potrebbero essere molto superiori a ```TcpConnection::$maxSendBufferSize```. Quando la dimensione del buffer di invio supera ```TcpConnection::$maxSendBufferSize```, se si continua a inviare dati tramite Connection::send(`$B`), i dati `$B` inviati questa volta non verranno inseriti nel buffer di invio, ma saranno scartati, attivando la callback `onError`.

In breve, finché il buffer di invio non è pieno, anche con un solo byte di spazio disponibile, la chiamata a Connection::send(```$A```) aggiungerà sicuramente ```$A``` al buffer di invio. Se, dopo l'inserimento nel buffer di invio, la dimensione del buffer supera il limite di ```TcpConnection::$maxSendBufferSize```, verrà attivata la callback onBufferFull.

## Parametri della funzione di callback

 ``` $connection ```

Oggetto di connessione, cioè un'istanza di [TcpConnection](../tcp-connection.md), utilizzato per operare sulla connessione del client, ad esempio [invio dei dati](../tcp-connection/send.md), [chiusura della connessione](../tcp-connection/close.md), ecc.

## Esempi

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Buffer pieno e non inviare di nuovo\n";
};
// Avvia il worker
Worker::runAll();
```

Nota: Oltre alla possibilità di utilizzare una funzione anonima come callback, è anche possibile utilizzare [altri metodi di callback](../faq/callback_methods.md) come indicato qui.

## Vedi anche
onBufferDrain: Attivato quando tutti i dati nel buffer di invio a livello di applicazione della connessione sono stati inviati completamente
