# __construct
## Description:
```php
void \Workerman\WebServer::__construct(string $address)
```
Create an WebServer instance

### Parameters
``` address ```

Address，such as ```tcp://0.0.0.0:80```

### Notice

These superglobal variables are available

``` $_SERVER、$_GET、$_POST、$_FILES、$_COOKIE、$_SESSION、$_REQUEST ```


**This WebServer's ```$_FILES``` is different from native php**

```php

var_export($_FILES);

// The above code will output like
array(
    0 => array(
        'file_name' => 'logo.png', // file name
        'file_size' => 23654,      // file size
        'file_data' => '*****',    // file data ,maybe binary
    ),
    1 => array(
        'file_name' => 'file.tar.gz',
        'file_size' => 128966,
        'file_data' => '*****',
    ),
    ...
);

```

Save upload files like
```php
foreach($_FILES as $file_info)
{
    file_put_contents('/tmp/'.$file_info['file_name'], $file_info['file_data']);
}
```



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

