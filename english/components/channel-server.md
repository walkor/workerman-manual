# Channel Component Server

**``` (Requires Workerman version >= 3.3.0) ```**

# __construct
```php
void \Channel\Server::__construct([string $listen_ip = '0.0.0.0', int $listen_port = 2206])
```

Instantiate a \Channel\Server server.

### Parameters
``` listen_ip ```

The local IP address to listen on. If not passed, the default is ```0.0.0.0```.

``` listen_port ```

The port to listen on. If not passed, the default is 2206.

## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// If no parameters are passed, it defaults to listening on 0.0.0.0:2206
$channel_server = new Channel\Server();

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```
