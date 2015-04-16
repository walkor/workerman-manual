# onWorkerStop
## 说明:
```php
callback Worker::$onWorkerStop
```

设置Workert停止时的回调函数，即当Worker收到stop信号后执行Worker::onWorkerStop指定的回调函数

## 回调函数的参数

``` $worker ```

即Worker对象

## 范例

```php
use WorkerMan\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStop = function($worker)
{
    echo "Worker stopping...\n";
};
```
