# IPv6

Вопрос: Как сделать возможным доступ клиента как по адресу IPv4, так и по адресу IPv6?

Ответ: При инициализации контейнера просто укажите прослушиваемый адрес как ```[::]```.

Например:
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
