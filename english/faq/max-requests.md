# How to set up Workerman to restart the current process after handling a certain number of requests
In order to streamline Workerman, this setting is not directly provided, but this functionality can be achieved through a few lines of code.
```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // Business logic implementation
    if(++$request_count > 10000) {
        // After reaching 10,000 requests, exit the current process, and the main process will automatically restart a new process
        Worker::stopAll();
    }
};

```
