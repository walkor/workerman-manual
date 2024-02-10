# workerman/crontab

# Description
`workerman/crontab` is a timing task program based on workerman, similar to linux's crontab. `workerman/crontab` supports second-level timing.

> Setting the time zone of PHP is required before using `workerman/crontab`, otherwise the running result may be different from the expectation.

## Time Description
```plaintext
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ day of week (0 - 6) (Sunday=0)
|   |   |   |   +------ month (1 - 12)
|   |   |   +-------- day of month (1 - 31)
|   |   +---------- hour (0 - 23)
|   +------------ min (0 - 59)
+-------------- sec (0-59) [Optional, if omitted, the minimum time granularity is minute]
```

# Installation
```plaintext
composer require workerman/crontab
```

# Example
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Set the time zone to avoid inconsistencies between the running results and expectations
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Execute at the 1st second of every minute.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Execute at 7:50 every day, note that the second digit is omitted here.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Note: Scheduled tasks will not be executed immediately. All scheduled tasks will start timing execution in the next minute.

# Interface
**Crontab::destroy()**

Destroy the timer
```php
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
