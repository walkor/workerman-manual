# onBufferFull
## Erklärung:
```php
Rückrufverbindung::$onBufferFull
```

Funktioniert genauso wie das Rückruf Worker::$onBufferFull, mit dem Unterschied, dass es nur für die aktuelle Verbindung gilt, d. h. der onBufferFull-Rückruf kann separat für eine bestimmte Verbindung festgelegt werden.
