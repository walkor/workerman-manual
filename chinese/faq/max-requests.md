# 如何设置workerman处理一定请求后重启当前进程
为了让workerman更加精简，并没有直接提供这个设置，不过可以通过几行代码实现该功能。
```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // 业务处理略
    if(++$request_count > 10000) {
        // 请求数达到10000后退出当前进程，主进程会自动重启一个新的进程
        Worker::stopAll();
    }
};

```