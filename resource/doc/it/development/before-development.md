# Lettura obbligatoria prima dello sviluppo

Per sviluppare un'applicazione con WorkerMan, è necessario comprendere i seguenti punti:


## I. Differenze tra lo sviluppo con WorkerMan e il normale sviluppo PHP

A parte l'impossibilità di utilizzare direttamente le variabili e le funzioni relative al protocollo HTTP, lo sviluppo con WorkerMan non differisce molto dal normale sviluppo PHP.

### 1. Differenze nel protocollo delle applicazioni
* Nello sviluppo PHP tradizionale si basa principalmente sul protocollo HTTP, il server Web ha già completato l'analisi del protocollo per lo sviluppatore.
* WorkerMan supporta vari protocolli, al momento ha incorporato i protocolli HTTP e WebSocket. Si consiglia agli sviluppatori di utilizzare protocolli personalizzati più semplici per la comunicazione.

 * Per lo sviluppo con protocollo HTTP fare riferimento alla sezione [Servizio Http](../http/request.md)

### 2. Differenze nel ciclo delle richieste
* Nell'applicazione Web PHP, una volta completata la richiesta, tutte le variabili e le risorse vengono rilasciate.
* Le applicazioni sviluppate con WorkerMan rimangono in memoria dopo il caricamento e l'analisi iniziale, quindi le definizioni delle classi, gli oggetti globali e i membri statici delle classi non vengono rilasciati e possono quindi essere riutilizzati più volte successivamente.

### 3. Attenzione a evitare la ridichiarazione delle classi e delle costanti
* Dato che WorkerMan memorizzerà i file PHP compilati, è necessario evitare di richiamare più volte gli stessi file di definizione di classi o costanti. Si consiglia di utilizzare require_once/include_once per caricare i file.

### 4. Attenzione al rilascio delle risorse di connessione in modalità singleton
* Poiché WorkerMan non rilascia gli oggetti globali e i membri statici delle classi dopo ogni richiesta, nelle modalità singleton come i database, è comune mantenere l'istanza del database (che include una connessione del socket del database) in un membro statico. Questo consente a WorkerMan di riutilizzare questa connessione del socket del database per l'intera durata della vita del processo. È importante notare che il server del database potrebbe chiudere attivamente la connessione del socket se non c'è attività per un certo periodo di tempo, e ciò potrebbe causare un errore quando si cerca di utilizzare nuovamente questa istanza del database (l'errore è simile a "il mysql è andato via"). WorkerMan fornisce una [classe per il database](../components/workerman-mysql.md) con una funzionalità di riconnessione, che i programmatori possono utilizzare direttamente.

### 5. Non utilizzare le istruzioni di uscita exit e die
* WorkerMan viene eseguito in modalità a riga di comando PHP, quindi l'uso delle istruzioni di uscita exit e die provocherà la chiusura del processo corrente. Anche se il processo figlio verrà immediatamente ricreato per continuare il servizio, potrebbe comunque avere un impatto sul business.

### 6. I cambiamenti nel codice richiedono il riavvio del servizio per essere effettivi
Poiché WorkerMan è in esecuzione nella memoria permanente, una volta caricata in memoria la definizione della classe PHP o delle funzioni, non verrà più letta dal disco. Pertanto, ogni volta che si modifica il codice dell'applicazione, è necessario riavviare il servizio per renderlo effettivo.


## II. Concetti di base da comprendere

### 1. Protocollo di trasporto TCP
Il TCP è un protocollo di trasporto basato su IP, orientato alla connessione e affidabile. Una caratteristica importante del protocollo TCP è che si basa sul flusso di dati, quindi le richieste inviate dal client al server arrivano ininterrottamente e il server può ricevere dati che non costituiscono una richiesta completa o più richieste concatenate. Pertanto, l'obiettivo del protocollo di livello applicazione è principalmente definire una serie di regole per delineare le richieste e evitare una confusione dei dati delle richieste.

### 2. Protocollo di livello applicazione
Il protocollo di livello applicazione (application layer protocol) definisce come i processi delle applicazioni su sistemi terminali diversi (client, server) si scambiano messaggi, ad esempio HTTP e WebSocket rientrano nella categoria dei protocolli di livello applicazione. Ad esempio, un semplice protocollo di livello applicazione potrebbe essere il seguente: ```{"module":"user","action":"getInfo","uid":456}\n"```. In questo protocollo, la fine della richiesta è segnata da ```"\n"``` (nota che ```"\n"``` rappresenta un ritorno a capo) e il corpo del messaggio è una stringa.

### 3. Connessione breve
Una connessione breve significa che viene stabilita una connessione ogni volta che ci sia un'interazione tra le due parti, e una volta completato l'invio dei dati, la connessione viene interrotta, ovvero ogni connessione viene utilizzata per inviare solo un'operazione commerciale. Questo tipo di connessione è comunemente utilizzato nei servizi HTTP dei siti Web.

*Lo sviluppo di un'applicazione con connessioni brevi può essere preso in considerazione nel flusso di sviluppo di base.*

### 4. Connessione a lungo termine
Una connessione a lungo termine, invece, consente di inviare più pacchetti di dati su una singola connessione.

Nota: Le applicazioni con connessioni a lungo termine devono utilizzare un [heartbeat](../faq/heartbeat.md), altrimenti, a causa della inattività prolungata, la connessione potrebbe essere interrotta dal firewall del nodo di routing.

Le connessioni a lungo termine sono ampiamente utilizzate in comunicazioni punto a punto con operazioni frequenti. Ogni connessione TCP richiede tre fasi di handshake, che richiedono tempo. Se ogni operazione richiede prima di tutto una connessione, allora la velocità di elaborazione sarà notevolmente rallentata. Pertanto, le connessioni a lungo termine vengono mantenute aperte dopo ogni operazione e i dati sono inviati direttamente in seguito, senza bisogno di stabilire una connessione TCP. Ad esempio, le connessioni con il database vengono tenute aperte in una connessione a lungo termine, poiché le comunicazioni frequenti con una connessione a breve durata possono causare errori di socket e uno spreco di risorse.

*Le applicazioni di sviluppo con connessioni a lungo termine possono essere prese in considerazione nel processo di sviluppo di Gateway/Worker.*

### 5. Riavvio graduale
Un riavvio normale comporta l'arresto di tutti i processi, seguito dalla creazione di nuovi processi di servizio. Durante questo processo, ci sarà un breve periodo in cui nessun processo fornirà il servizio esterno, il che potrebbe causare temporaneamente l'inaccessibilità del servizio, cosa che in caso di alto traffico inevitabilmente porta a un fallimento delle richieste.

Al contrario, un riavvio graduale comporta non l'interruzione di tutti i processi contemporaneamente, ma viene arrestato un processo alla volta, sostituendolo immediatamente con un nuovo, fino a quando tutti i vecchi processi non vengono sostituiti.

WorkerMan consente un riavvio graduale con il comando ```php your_file.php reload```, permettendo di aggiornare l'applicazione senza compromettere la qualità del servizio.

**Nota: solo i file caricati automaticamente tramite il callback on{...} verranno aggiornati automaticamente dopo un riavvio graduale. I file caricati direttamente nello script di avvio o il codice codificato non saranno aggiornati automaticamente dopo il reload.**


## III. Differenza tra processo principale e processo figlio
È importante notare se il codice viene eseguito nel processo principale o in quello figlio. In generale, tutto il codice eseguito prima della chiamata a ```Worker::runAll();``` viene eseguito nel processo principale, mentre il codice eseguito nel callback onXXX appartiene al processo figlio. È importante notare che il codice scritto dopo ```Worker::runAll();``` non verrà eseguito.

Ad esempio, nel seguente codice:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Eseguito nel processo principale
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// L'assegnazione avviene nel processo principale
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Questa parte viene eseguita nel processo figlio
    $connection->send('hello ' . $data);
};

Worker::runAll();
```

**Nota:** Non è consigliato inizializzare le risorse di connessione come database, memcache e redis nel processo principale, poiché le connessioni inizializzate nel processo principale potrebbero essere ereditate automaticamente dai processi figlio (specialmente utilizzando singleton). Tutti i processi condividerebbero quindi una singola connessione, e i dati restituiti dal server attraverso questa connessione sarebbero leggibili su più processi, causando una confusione dei dati. Allo stesso modo, se qualsiasi processo chiude la connessione (ad esempio, quando si esegue il demone, il processo principale esce e chiude la connessione), tutte le connessioni dei processi figlio verranno chiuse insieme, causando errori imprevisti come ad esempio "mysql è andato via". Si raccomanda di inizializzare le risorse di connessione nell'onWorkerStart.

Spero che questo ti sia utile! Fammi sapere se hai bisogno di altre traduzioni.
