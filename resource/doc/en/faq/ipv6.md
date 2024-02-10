# IPv6

Question: How can clients access the server using both IPv4 and IPv6 addresses?

Answer: Use `[::]` as the listening address when initializing the container.

For example:
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
