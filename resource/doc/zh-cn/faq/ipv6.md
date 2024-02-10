# ipv6

问：如何让客户端即能通过ipv4地址访问，也能通过ipv6地址访问?

答：在初始化容器的时候监听地址写```[::]```即可。

例如
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```


