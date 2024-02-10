# send
## Descrizione:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Invia dati al client

## Parametri

 ``` $data ```

I dati da inviare, se è stato specificato un protocollo durante l'inizializzazione della classe Worker, verrà automaticamente chiamato il metodo di codifica del protocollo per completare il lavoro di impacchettamento del protocollo e inviarlo al client

 ``` $raw ```
 
Indica se inviare i dati grezzi, cioè senza chiamare il metodo di codifica del protocollo. Il valore predefinito è false, quindi viene chiamato automaticamente il metodo di codifica del protocollo

## Valore restituito

true indica che i dati sono stati scritti con successo nel buffer di invio del socket a livello di sistema per quella connessione

null indica che i dati sono stati scritti nel buffer di invio a livello di applicazione per quella connessione, in attesa di essere scritti nel buffer di invio del socket a livello di sistema

false indica un fallimento nell'invio, il motivo potrebbe essere che la connessione del client è stata chiusa o che il buffer di invio a livello di applicazione per quella connessione è pieno

## Nota
Il valore restituito di send ```true``` indica solamente che i dati sono stati scritti con successo nel buffer di invio del socket a livello di sistema per quella connessione e non significa che i dati sono stati inviati con successo al buffer di ricezione del socket di destinazione, né tanto meno che l'applicazione di destinazione ha già letto i dati dal buffer di ricezione del socket locale. **Tuttavia, anche in questo caso, purché send non restituisca false e la connessione di rete non sia stata interrotta, e il client riceva normalmente, i dati possono essere considerati di essere stati inviati al destinatario al 100%.** 

Poiché i dati nel buffer di invio del socket vengono inviati asincronamente dal sistema operativo al socket di destinazione, il sistema operativo non fornisce un meccanismo di conferma all'applicazione, quindi l'applicazione non può sapere quando i dati nel buffer di invio del socket inizieranno ad essere inviati e nemmeno se i dati nel buffer di invio del socket sono stati inviati con successo. Per questi motivi, Workerman non può fornire direttamente un'interfaccia di conferma dei messaggi.

Se il business richiede di garantire che ciascun messaggio venga ricevuto dal client, è possibile aggiungere un meccanismo di conferma sul business. Il meccanismo di conferma può variare a seconda del business, anche se lo stesso business può avere più metodi per il meccanismo di conferma.

Ad esempio, un sistema di chat può utilizzare un meccanismo di conferma come questo. Salvare ogni messaggio nel database e ogni messaggio ha un campo per indicare se è stato letto. Ogni volta che il client riceve un messaggio, invia un pacchetto di conferma al server e il server imposta il relativo messaggio come letto. Quando il client si connette al server (di solito quando l'utente effettua il login o si riconnette dopo una disconnessione), controlla se ci sono messaggi non letti nel database e, in caso affermativo, li invia al client. Allo stesso modo, quando il client riceve un messaggio, notifica il server che è stato letto. In questo modo è possibile garantire che ogni messaggio venga ricevuto dall'altra parte. Naturalmente, lo sviluppatore può creare la propria logica di conferma.

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Verrà chiamato automaticamente \Workerman\Protocols\Websocket::encode per imballare i dati nel protocollo websocket e inviarli
    $connection->send("hello\n");
};
// Esegui il worker
Worker::runAll();
```
