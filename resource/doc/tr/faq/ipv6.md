# ipv6

Soru: Müşterinin hem ipv4 adresi üzerinden hem de ipv6 adresi üzerinden erişebilmesi için ne yapılabilir?

Cevap: Konteyneri başlatırken dinleme adresini ```[::]``` olarak ayarlayın.

Örneğin
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
