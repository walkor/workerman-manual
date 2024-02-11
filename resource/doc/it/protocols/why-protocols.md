# Ruolo dei protocolli di comunicazione
Poiché TCP si basa sul flusso, i dati di richiesta inviati dal client scorrono verso il server come un flusso d'acqua. Una volta che il server rileva l'arrivo dei dati, deve verificare se i dati sono completi, in quanto potrebbe essere arrivata solo una parte della richiesta al server, o addirittura potrebbero essere arrivate più richieste concatenate. Per determinare se una richiesta è stata completamente ricevuta o separare le richieste concatenate, è necessario definire un set di protocolli di comunicazione.

## Perché definire un protocollo in Workerman?
Lo sviluppo tradizionale in PHP si basa principalmente su HTTP, e il parsing e la gestione dei protocolli HTTP sono principalmente affidati al WebServer, quindi gli sviluppatori non devono preoccuparsi degli aspetti relativi ai protocolli. Tuttavia, quando si sviluppa basandosi su protocolli diversi da HTTP, gli sviluppatori devono prendere in considerazione proprio questi aspetti.

## Protocolli supportati da Workerman
Al momento, Workerman supporta già i protocolli HTTP, websocket, text (vedi spiegazione in appendice) e frame (vedi spiegazione in appendice), oltre al protocollo ws (vedi spiegazione in appendice). Quando si necessita di comunicare basandosi su tali protocolli, è possibile utilizzarli direttamente specificando il protocollo durante l'inizializzazione del Worker, come mostrato di seguito:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 indica di usare il protocollo websocket per ascoltare sulla porta 2345
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// Protocollo text
$text_worker = new Worker('text://0.0.0.0:2346');

// Protocollo frame
$frame_worker = new Worker('frame://0.0.0.0:2347');

// Worker TCP, trasmissione diretta tramite socket, senza l'utilizzo di alcun protocollo a livello di applicazione
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// Worker UDP, senza l'utilizzo di alcun protocollo a livello di applicazione
$udp_worker = new Worker('udp://0.0.0.0:2349');

// Worker del dominio Unix, senza l'utilizzo di alcun protocollo a livello di applicazione
$unix_worker = new Worker('unix:///tmp/wm.sock');
```

## Utilizzo di un protocollo di comunicazione personalizzato
Quando i protocolli di comunicazione predefiniti di Workerman non soddisfano le esigenze di sviluppo, gli sviluppatori possono personalizzare il proprio protocollo di comunicazione, come illustrato nella sezione successiva.

**Suggerimento:**
Workerman include nativamente un protocollo di testo, con il formato testo + carattere di nuova riga. Il protocollo text semplifica molto lo sviluppo e il debug, ed è adatto alla maggior parte delle esigenze di protocolli personalizzati, supportando anche il debug tramite telnet. Se gli sviluppatori devono sviluppare il proprio protocollo di applicazione, possono utilizzare direttamente il protocollo text senza doverlo sviluppare separatamente.

Consulta la sezione "Appendice: Protocollo text" per maggiori dettagli sul protocollo text.
