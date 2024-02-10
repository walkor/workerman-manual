# Flusso Base
(Prendiamo come esempio un server di chat WebSocket semplice)

#### 1. Creare una directory del progetto in qualsiasi posizione
Ad esempio: SimpleChat/
Entrare nella directory e eseguire `composer require workerman/workerman`

#### 2. Includere `vendor/autoload.php` (generato dopo l'installazione di Composer)
Creare start.php e includere `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Scegliere il protocollo
Qui scegliamo il protocollo di testo (un protocollo personalizzato in WorkerMan, con formato testo + nuova riga)

(Attualmente WorkerMan supporta HTTP, WebSocket, protocollo di testo. Se è necessario utilizzare un altro protocollo, consultare il capitolo sui protocolli per sviluppare il proprio protocollo)

#### 4. Scrivere lo script di avvio dell'ingresso in base alle esigenze
Di seguito viene mostrato un semplice file di ingresso per una chat room.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Quando un client si connette, assegna un uid, salva la connessione e notifica tutti i client
function gestisci_connessione($connessione)
{
    global $text_worker, $global_uid;
    // Assegna un uid a questa connessione
    $connessione->uid = ++$global_uid;
}

// Quando un client invia un messaggio, lo inoltra a tutti
function gestisci_messaggio(TcpConnection $connessione, $dati)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("utente[{$connessione->uid}] ha detto: $dati");
    }
}

// Quando un client si disconnette, invia una notifica a tutti i client
function gestisci_chiusura($connessione)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("utente[{$connessione->uid}] si è disconnesso");
    }
}

// Crea un Worker con protocollo di testo in ascolto sulla porta 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// Avvia solo 1 processo per facilitare lo scambio di dati tra i client
$text_worker->count = 1;

$text_worker->onConnect = 'gestisci_connessione';
$text_worker->onMessage = 'gestisci_messaggio';
$text_worker->onClose = 'gestisci_chiusura';

Worker::runAll();
```

#### 5. Testing
Il protocollo di testo può essere testato tramite il comando telnet
```shell
telnet 127.0.0.1 2347
```
