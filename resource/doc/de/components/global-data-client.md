# GlobalData Komponenten Client
**``` (Erfordert Workerman-Version >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Instanziert ein \GlobalData\Client-Clientobjekt. Durch das Zuweisen von Attributen an das Clientobjekt können Daten zwischen Prozessen ausgetauscht werden.

### Parameter
GlobalData-Server Serveradresse im Format ```<IP-Adresse>:<Port>```, z.B. ```127.0.0.1:2207```.

Bei einem GlobalData-Server-Cluster wird ein Arrays von Adressen übergeben, z.B. ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```.

## Erklärung
Unterstützt Zuweisungen, Lesen, isset-Prüfungen und unset-Operationen.
Unterstützt gleichzeitig CAS-Atomoperationen.

## Beispiel

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// GlobalData Server
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// Beim Start des Prozesses
$worker->onWorkerStart = function()
{
    // Initialisiere einen globalen global data client
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// Jedes Mal, wenn der Server eine Nachricht empfängt
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Ändere den Wert von $global->somedata. Andere Prozesse teilen diese Variable $global->somedata.
    global $global;
    echo "jetzt global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Alle Verwendungen (auch in der PHP-FPM-Umgebung möglich)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));
```

## Hinweis:
Der GlobalData-Komponente kann keine Ressourcentypdaten teilen, wie z.B. MySQL-Verbindungen, Socket-Verbindungen, usw.

Wenn GlobalData/Client in einer Workerman-Umgebung verwendet wird, instanziieren Sie das GlobalData/Client-Objekt in den onXXX-Callback-Funktionen, z.B. in onWorkerStart.

Variablen können nicht wie folgt gemeinsam genutzt werden.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Stattdessen können sie wie folgt verwendet werden.
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
