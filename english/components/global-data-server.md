# GlobalData Component Server
**``` (Requires Workerman version >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2207])
```

Instantiates a \GlobalData\Server service.

### Parameters
``` listen_ip ```

The local IP address to listen on. If not passed, the default is ```0.0.0.0```.

``` listen_port ```

The port to listen on. If not passed, the default is 2207.

## Example
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Listen on a port
$worker = new GlobalData\Server('127.0.0.1', 2207);

Worker::runAll();
```
