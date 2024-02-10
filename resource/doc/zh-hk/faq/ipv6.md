# IPv6

問：如何使客戶端能夠通過IPv4地址訪問，也能通過IPv6地址訪問？

答：在初始化容器的時候，設置監聽地址為```[::]```即可。

例如
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
