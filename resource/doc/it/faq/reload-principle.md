# Principio del riavvio graduale
## Cosa si intende per riavvio graduale?

Il riavvio graduale, diversamente dal riavvio normale, consente di riavviare i servizi (di solito si riferisce ai servizi con connessioni brevi) senza influenzare gli utenti, al fine di ricaricare il programma PHP e completare l'aggiornamento del codice aziendale.

Il riavvio graduale è generalmente utilizzato durante l'aggiornamento dell'azienda o durante il processo di distribuzione della versione, in modo da evitare l'impatto temporaneo della disponibilità del servizio provocato dal riavvio dei servizi a causa dell'aggiornamento del codice.

> **Nota**
> Il sistema Windows non supporta il reload.

> **Nota**
> Per i servizi con connessioni persistenti (ad esempio websocket), le connessioni vengono interrotte durante il riavvio graduale del processo. La soluzione consiste nell'utilizzare un'architettura simile a [gatewayWorker](https://www.workerman.net/doc/gateway-worker), in cui un gruppo di processi mantiene le connessioni e imposta la proprietà [reloadable](../worker/reloadable.md) di questo gruppo di processi su false. La logica aziendale avvia un altro gruppo di processi worker per la gestione e la comunicazione tra gateway e processi worker avviene tramite TCP. Quando è necessario apportare modifiche all'attività, è sufficiente riavviare i processi worker.

## Limitazioni
**Nota: Solo i file caricati nei callback on{...} verranno aggiornati automaticamente dopo il riavvio graduale. I file caricati direttamente nello script di avvio o il codice rigido eseguito durante il reload non verranno aggiornati automaticamente.**

#### Il seguente codice non verrà aggiornato dopo il reload
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('Ciao'); // Il codice rigido non supporta l'aggiornamento in tempo reale
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // I file caricati direttamente nello script di avvio non supportano l'aggiornamento in tempo reale
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Supponendo che la classe MessageHandler abbia un metodo onMessage
```


#### Il seguente codice verrà aggiornato automaticamente dopo il reload
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart è il callback attivato dopo l'avvio del processo
    require_once __DIR__ . '/your/path/MessageHandler.php'; // Il file caricato dopo l'avvio del processo supporta l'aggiornamento in tempo reale
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
Dopo aver apportato modifiche a MessageHandler.php, eseguendo `php start.php reload`, MessageHandler.php verrà ricaricato in memoria per riflettere l'aggiornamento della logica aziendale.


> **Nota**
> Nel codice sopra, per comodità, è stato utilizzato l'istruzione `require_once`. Se il tuo progetto supporta il caricamento automatico PSR-4, non è necessario utilizzare l'istruzione `require_once`.

## Principio del riavvio graduale

Workerman consiste in un processo principale e processi figlio, il processo principale si occupa di monitorare i processi figlio, mentre i processi figlio gestiscono le connessioni dei client e i dati delle richieste inviate, elaborando le richieste e restituendo i dati ai client. Quando il codice aziendale viene aggiornato, è sufficiente aggiornare i processi figlio per aggiornare il codice.

Quando il processo principale di Workerman riceve un segnale di riavvio graduale, invia un segnale di uscita sicura (per consentire al processo corrispondente di gestire la richiesta corrente prima di uscire) a uno dei processi figlio. Quando il processo esce, il processo principale crea un nuovo processo figlio (che include il nuovo codice PHP) e quindi invia nuovamente il comando di stop a un altro vecchio processo, questo processo uno per uno si arresta e si ricrea, finché tutti i vecchi processi non vengono sostituiti.

Vediamo che il riavvio graduale è effettivamente realizzato facendo uscire uno per uno i vecchi processi aziendali e creandone di nuovi. Per evitare l'impatto sugli utenti durante il riavvio graduale, è necessario che i processi non conservino informazioni di stato relative agli utenti, quindi è meglio che i processi aziendali siano senza stato, per evitare la perdita di informazioni a causa dell'uscita del processo.
