# วิธีการสร้างบริการ UDP

การสร้างบริการ UDP ใน workerman จะง่ายมาก โดยคล้ายกับโค้ดตัวอย่างด้านล่าง

```php
$udp_worker = new Worker('udp://0.0.0.0:9292');
$udp_worker->onMessage = function($connection, $data){
    var_dump($data);
    $connection->send('get');
};
Worker::runAll();
```

โปรดทราบ: เนื่องจาก UDP ไม่มีการเชื่อมต่อ ดังนั้นบริการ UDP จะไม่มีเหตุการณ์ onConnect และ onClose 
