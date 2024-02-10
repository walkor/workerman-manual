# Composant client GlobalData
**``` (Exige Workerman version >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Instancie un objet client \GlobalData\Client. Cela permet le partage de données entre les processus en affectant des propriétés à l'objet client.

### Paramètres
Adresse du serveur GlobalData, au format ```<adresse IP>:<port>```, par exemple ```127.0.0.1:2207```.

Si c'est un cluster de serveurs GlobalData, passez un tableau d'adresses, par exemple ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```.

## Description
Prend en charge l’affectation de valeurs, la lecture, isset, unset.
Prend également en charge l'opération atomique CAS.


## Exemples

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Serveur GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// Au démarrage du processus
$worker->onWorkerStart = function()
{
    // Initialise un client global data
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// À chaque fois que le serveur reçoit un message
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Change la valeur de $global->somedata, les autres processus partageront cette variable $global->somedata
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Utilisation complète (également possible dans un environnement php-fpm)
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

## Remarque :
Le composant GlobalData ne peut pas partager de données de type ressource, telles que les connexions mysql, les connexions de socket, etc.

Si vous utilisez GlobalData/Client dans un environnement Workerman, veuillez instancier l'objet GlobalData/Client dans les rappels onXXX, par exemple dans onWorkerStart.

Il ne faut pas opérer sur les variables partagées de cette manière.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Vous pouvez le faire de cette manière
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
