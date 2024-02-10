# Descrizione
A partire dalla versione 4.x, Workerman ha potenziato il supporto ai servizi HTTP, introducendo la classe delle richieste, la classe delle risposte, la classe delle sessioni e l'[SSE](SSE.md). Se si desidera utilizzare il servizio HTTP di Workerman, è fortemente consigliato utilizzare la versione 4.x o successiva.

**Si noti che quanto segue si riferisce alla versione 4.x di Workerman e non è compatibile con la versione 3.x.**

## Cambiare il motore di archiviazione della sessione
Workerman offre il supporto per l'archiviazione delle sessioni in file e nel database Redis. L'archiviazione predefinita avviene tramite file. Se si desidera passare all'archiviazione sui database Redis, si prega di fare riferimento al codice seguente.
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// Configurazione di Redis
$config = [
    'host'     => '127.0.0.1', // parametro obbligatorio
    'port'     => 6379,        // parametro obbligatorio
    'timeout'  => 2,           // parametro facoltativo
    'auth'     => '******',    // parametro facoltativo
    'database' => 1,           // parametro facoltativo
    'prefix'   => 'session_'   // parametro facoltativo
];
// Utilizzare il metodo Workerman\Protocols\Http\Session::handlerClass per cambiare la classe del motore di archiviazione della sessione
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Impostare la posizione di archiviazione della sessione
Quando si utilizza l'archiviazione predefinita, i dati delle sessioni sono salvati per impostazione predefinita sul disco, nella posizione restituita da `session_save_path()`. È possibile utilizzare il seguente metodo per modificare la posizione di archiviazione.
```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Impostare la posizione di archiviazione dei file delle sessioni
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Eseguire il worker
Worker::runAll();
```

## Pulizia dei file di sessione
Utilizzando l'archiviazione predefinita delle sessioni, sul disco verranno creati molti file di sessione. Workerman pulirà i file di sessione scaduti in base alle impostazioni `session.gc_probability`, `session.gc_divisor` e `session.gc_maxlifetime` presenti nel file php.ini. Per ulteriori informazioni su queste impostazioni, fare riferimento al [manuale di PHP](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability).

## Cambiare il motore di archiviazione
Oltre all'archiviazione delle sessioni tramite file e Redis, Workerman consente di aggiungere nuovi motori di archiviazione delle sessioni implementando l'interfaccia standard [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php), come ad esempio l'archiviazione delle sessioni su MongoDB o MySQL.

**Procedure per aggiungere un nuovo motore di archiviazione delle sessioni**
  1. Implementare l'interfaccia [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php)
  2. Utilizzare il metodo `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` per sostituire l'interfaccia del gestore della sessione sottostante.

**Implementare l'interfaccia SessionHandlerInterface**

Il gestore personalizzato della sessione deve implementare l'interfaccia SessionHandlerInterface, che comprende i seguenti metodi:
```php
SessionHandlerInterface {
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**Spiegazione di SessionHandlerInterface**
 - Il metodo read viene utilizzato per leggere tutti i dati di sessione corrispondenti a session_id dallo storage. Non è necessario eseguire operazioni di deserializzazione dei dati, poiché il framework lo farà automaticamente.
 - Il metodo write serve per scrivere i dati di sessione corrispondenti a session_id nello storage. Non è necessario eseguire operazioni di serializzazione dei dati, poiché il framework lo farà già in modo automatico.
 - Il metodo destroy serve per distruggere i dati di sessione corrispondenti a session_id.
 - Il metodo gc serve per eliminare i dati di sessione scaduti. Lo storage dovrebbe eliminare tutte le sessioni con un timestamp di ultima modifica superiore a maxlifetime.
 - Il metodo close non richiede alcuna operazione e deve semplicemente restituire true.
 - Il metodo open non richiede alcuna operazione e deve semplicemente restituire true.

**Sostituire il gestore sottostante**

Dopo aver implementato l'interfaccia SessionHandlerInterface, utilizzare il seguente metodo per modificare il gestore sottostante della sessione.
```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name è il nome della classe del gestore SessionHandler che implementa l'interfaccia SessionHandlerInterface. Se la classe è all'interno di un namespace, è necessario specificare il namespace completo.
 - $config sono i parametri del costruttore della classe SessionHandler.

**Implementazione specifica**

*Si noti che la classe MySessionHandler è solo un esempio per illustrare il processo di modifica del gestore sottostante della sessione e non deve essere utilizzata in un ambiente di produzione.*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

// Si supponga che la nuova classe MySessionHandler richieda alcuni parametri di configurazione
$config = ['host' => 'localhost'];
// Utilizzare il metodo Workerman\Protocols\Http\Session::handlerClass($class_name, $config) per cambiare la classe del gestore sottostante della sessione
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
