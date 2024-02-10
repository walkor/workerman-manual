# ipv6

Frage: Wie kann ein Client sowohl über eine IPv4-Adresse als auch über eine IPv6-Adresse zugreifen?

Antwort: Sie können dies erreichen, indem Sie bei der Initialisierung des Containers die Adresse auf ```[::]``` setzen.

Beispiel:
```php
$worker = new Worker('http://[::]:8080');
$gateway = new Gateway('websocket://[::]:8081');
```
