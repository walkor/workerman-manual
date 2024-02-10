# Development Specification

## Application Directory

The application directory can be placed anywhere.

## Entry File

Similar to PHP applications under nginx+PHP-FPM, Workerman applications also require an entry file, and the entry file is run in PHP Cli mode.

The entry file contains the code related to creating listening processes. For example, the following code snippet is based on Worker development:

test.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Create a Worker to listen on port 2345 using the http protocol
$http_worker = new Worker("http://0.0.0.0:2345");

// Start 4 processes to provide service externally
$http_worker->count = 4;

// Reply with hello world to the browser when receiving data from the browser
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Send hello world to the browser
    $connection->send('hello world');
};

Worker::runAll();
```

## Code Specification in Workerman

1. Class names are written in UpperCamelCase, and the class file name must be the same as the class name inside the file for automatic loading. For example:
```php
class UserInfo
{
...
```

2. Use namespaces where the namespace name corresponds to the directory path, based on the developer's project root directory. For example, in the project MyApp/, the class file MyApp/MyClass.php is in the project root directory, so the namespace is omitted. The class file MyApp/Protocols/MyProtocol.php is located in the Protocols directory of the MyApp project, so it needs to add the namespace `namespace Protocols;` as follows:
```php
namespace Protocols;
class MyProtocol
{
....
```

3. Use lowercase with underscores for regular functions and variables. For example:
```php
$connection_list = array();
function get_connection_list()
{
....
```

4. Use lowerCamelCase for class members and methods. For example:
```php
public $connectionList;
public function getConnectionList();
```

5. Use lowercase with underscores for function and class parameters. For example:
```php
function get_connection_list($one_param, $two_param)
{
....
```
