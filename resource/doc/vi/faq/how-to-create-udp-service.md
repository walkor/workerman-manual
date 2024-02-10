# Cách thiết lập dịch vụ UDP

Trong workerman, việc thiết lập dịch vụ UDP rất đơn giản, tương tự như đoạn mã sau

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

Lưu ý: Vì UDP không có kết nối, nên dịch vụ UDP không có sự kiện onConnect và onClose.
