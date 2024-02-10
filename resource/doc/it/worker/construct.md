# Costruttore __construct

## Descrizione:
```php
Worker::__construct([string $listen , array $context])
```

Inizializza un'istanza del contenitore Worker, può impostare alcune proprietà del contenitore e interfacce di callback per completare funzionalità specifiche.

## Parametri
#### **``` $listen ```** (parametro opzionale, se omesso significa che non si sta in ascolto su nessuna porta)

Se si imposta il parametro di ascolto ```$listen```, verrà eseguito l'ascolto del socket.

Il formato di $listen è <protocollo>://<indirizzo di ascolto>

**<protocollo> può essere nei seguenti formati:** 

tcp: ad esempio ```tcp://0.0.0.0:8686```

udp: ad esempio ```udp://0.0.0.0:8686```

unix: ad esempio ```unix:///tmp/my_file ``` ```(richiede Workerman>=3.2.7)```

http: ad esempio ```http://0.0.0.0:80```

websocket: ad esempio ```websocket://0.0.0.0:8686```

text: ad esempio ```text://0.0.0.0:8686``` ```(text è un protocollo di testo integrato in Workerman, compatibile con telnet, vedere dettagli nella sezione Protocollo di testo dell'appendice)```

e altri protocolli personalizzati, vedere la sezione del manuale Protocolli di comunicazione personalizzati

**<indirizzo di ascolto> può essere nei seguenti formati:** 

Se si tratta di un socket Unix, l'indirizzo è il percorso locale di un file sul disco.

Se non è un socket Unix, il formato dell'indirizzo è <IP del computer locale>:<numero di porta>

<IP del computer locale> può essere ```0.0.0.0``` per ascoltare su tutte le interfacce di rete del computer locale, compreso l'IP interno ed esterno e l'anello locale 127.0.0.1

Se l'indirizzo IP del computer locale è ```127.0.0.1```, verrà eseguito l'ascolto sull'anello locale, accessibile solo dal computer locale e non dall'esterno.

Se l'indirizzo IP del computer locale è un IP interno, simile a ```192.168.xx.xx```, indica che verrà effettuato l'ascolto solo sull'IP interno e gli utenti esterni non potranno accedere.

Se il valore dell'indirizzo IP del computer locale non corrisponde a un IP locale, non sarà possibile effettuare l'ascolto e verrà visualizzato l'errore "Cannot assign requested address"

**Nota:** Il <numero di porta> non può essere maggiore di 65535. Se il <numero di porta> è inferiore a 1024, è necessario disporre dei privilegi di root per ascoltare. La porta in ascolto deve essere una porta non utilizzata nel computer locale, in caso contrario non sarà possibile effettuare l'ascolto e verrà visualizzato l'errore "Address already in use"

#### **``` $context ```**

Un array utilizzato per passare le opzioni di contesto del socket. Vedere [Opzioni di contesto del socket](https://php.net/manual/it/context.socket.php)

## Esempi

Worker come contenitore http per l'ascolto e la gestione delle richieste http
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

// Esegui il worker
Worker::runAll();
```

Worker come contenitore websocket per l'ascolto e la gestione delle richieste websocket
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Esegui il worker
Worker::runAll();
```

Worker come contenitore tcp per l'ascolto e la gestione delle richieste tcp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Esegui il worker
Worker::runAll();
```

Worker come contenitore udp per l'ascolto e la gestione delle richieste udp
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Esegui il worker
Worker::runAll();
```

Worker in ascolto su un socket di dominio Unix ```(richiede Workerman versione>=3.2.7)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// Esegui il worker
Worker::runAll();
```

Contenitore Worker che non effettua alcun ascolto, utilizzato per gestire determinati compiti periodici
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // Esegui ogni 2,5 secondi
    $intervallo_di_tempo = 2.5;
    Timer::add($intervallo_di_tempo, function()
    {
        echo "task run\n";
    });
};

// Esegui il worker
Worker::runAll();
```

**Worker in ascolto su una porta con un protocollo personalizzato**

Struttura finale delle directory
```
├── Protocols              // Questa è la directory Protocols da creare
│   └── MyTextProtocol.php // Questo è il file di protocollo personalizzato da creare
├── test.php  // Questo è lo script test da creare
└── Workerman // Directory del codice sorgente di Workerman, il codice al suo interno non deve essere modificato
```

1. Creare la directory Protocols e creare un file di protocollo
Protocols/MyTextProtocol.php (fare riferimento alla struttura della directory sopra)

```php
// Spazio dei nomi del protocollo personalizzato degli utenti unificato come Protocols
namespace Protocols;
// Protocollo testuale semplice, il formato del protocollo è testo + nuova riga
class MyTextProtocol
{
    // Funzione di suddivisione del pacchetto, restituire la lunghezza del pacchetto corrente
    public static function input($recv_buffer)
    {
        // Trovare il carattere di nuova riga
        $pos = strpos($recv_buffer, "\n");
        // Se non viene trovato il carattere di nuova riga, significa che non è un pacchetto completo, restituisce 0 per continuare ad attendere i dati
        if($pos === false)
        {
            return 0;
        }
        // Se viene trovato il carattere di nuova riga, restituisce la lunghezza del pacchetto corrente, inclusa la nuova riga
        return $pos+1;
    }

    // Dopo aver ricevuto un pacchetto completo, eseguire automaticamente il decode tramite decode('dati ricevuti')
    // I risultati vengono passati a $data tramite il callback onMessage
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // Prima di inviare dati al client, vengono automaticamente codificati tramite encode, e quindi inviati al client, qui è stata aggiunta la nuova riga
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. Utilizzare il protocollo MyTextProtocol per ascoltare e gestire le richieste

Seguire la struttura delle directory della struttura finale

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### Worker protocol personalizzato ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * Dopo aver ricevuto dati completi (terminati da una nuova riga), eseguire automaticamente MyTextProtocol::decode('dati ricevuti')
 * I risultati vengono passati a $data tramite il callback onMessage
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * Invia dati al client, automaticamente chiamerà MyTextProtocol::encode('hello world') per codificare il protocollo,
     * quindi invierà al client
     */
    $connection->send("hello world");
};

// Esegui tutti i worker
Worker::runAll();
```

3. Testing

Aprire il terminale, accedere alla directory in cui si trova test.php e digitare ```php test.php start```

```sh
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

Aprire un terminale e utilizzare telnet per testare (si consiglia di utilizzare il telnet di sistema di Linux)

Assumendo un test locale,
Eseguire il telnet 127.0.0.1 5678 sul terminale
Quindi inserire hi e premere Invio
Verrà ricevuto il messaggio di "hello world\n"
```sh
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world

```
