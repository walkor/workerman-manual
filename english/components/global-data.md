# GlobalData Variable Sharing Component
**```(Requires Workerman version >= 3.3.0)```**

Source code address: https://github.com/walkor/GlobalData

## Note
GlobalData requires Workerman version >= 3.3.0

## Installation

`composer require workerman/globaldata`

## Principle

Utilize the PHP magic methods ```__set __get __isset __unset``` to trigger communication with the GlobalData server, and the actual variable is stored on the GlobalData server. For example, when setting a non-existent property for a client class, the ```__set``` magic method is triggered. The client class sends a request to the GlobalData server in the ```__set``` method to store a variable. When accessing a non-existent variable of the client class, the class's ```__get``` method is triggered, and the client sends a request to the GlobalData server to read this value, thereby achieving inter-process variable sharing.

```php
require_once __DIR__ . '/vendor/autoload.php';

// Connect to the Global Data server
$global = new GlobalData\Client('127.0.0.1:2207');

// Trigger $global->__isset('somedata') to query the server if the value for the key somedata is stored
isset($global->somedata);

// Trigger $global->__set('somedata',array(1,2,3)) to notify the server to store the value array(1,2,3) for somedata
$global->somedata = array(1,2,3);

// Trigger $global->__get('somedata') to query the value corresponding to somedata from the server
var_export($global->somedata);

// Trigger $global->__unset('somedata') to notify the server to delete somedata and its corresponding value
unset($global->somedata);
```
