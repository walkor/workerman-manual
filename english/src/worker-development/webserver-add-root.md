# addRoot

## Description:
```php
int \Workerman\WebServer::addRoot(string $domain, string $path)
```
Set the document root directory for the ```$domain```


### Parameters
``` domain ```

Domain.


``` path ```

the document root directory for the ```$domain```


### Return Values
No value is returned.

### Examples
```php
use \Workerman\WebServer;
require_once './Workerman/Autoloader.php';

$webserver = new WebServer('http://0.0.0.0:80');
$webserver->addRoot('www.example.com', '/your/path/of/web/');
$sebserver->count = 4;

// Run all workers
Worker::runAll();
```
