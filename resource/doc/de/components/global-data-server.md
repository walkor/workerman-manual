# GlobalData-Komponente auf Serverseite
**```(Erfordert Workerman-Version >= 3.3.0)```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Instanziiert einen \GlobalData\Server-Dienst.

### Parameter
 ```listen_ip```

Die IP-Adresse des lokalen Hosts, standardmäßig ist es ```0.0.0.0```.

 ```listen_port```

Der zu überwachende Port, standardmäßig ist es 2207.


## Beispiel
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Port überwachen
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
