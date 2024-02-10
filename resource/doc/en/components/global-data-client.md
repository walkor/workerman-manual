# GlobalData Component Client
**``` (Requires Workerman version >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Instantiate a \GlobalData\Client client object. Share data between processes by assigning properties on the client object.

### Parameters
GlobalData server server address in the format ```<ip address>:<port>```, for example ```127.0.0.1:2207```.

If it is a GlobalData server cluster, pass in an array of addresses, for example ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```

## Description
Supports assignment, reading, isset, and unset operations.
Also supports CAS atomic operations.

## Example

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// GlobalData Server
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// When the process starts
$worker->onWorkerStart = function()
{
    // Initialize a global global data client
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// When the server receives a message
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Change the value of $global->somedata, and other processes will share this $global->somedata variable
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### All usage (can also be used in php-fpm environment)
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

## Note:
The GlobalData component cannot share data of resource types, such as mysql connections, socket connections, etc.

If using GlobalData/Client in the Workerman environment, please instantiate the GlobalData/Client object in the onXXX callback, such as instantiating it in onWorkerStart.

Cannot operate shared variables like this.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
It can be done like this
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
