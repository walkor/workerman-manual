# onBufferDrain
## Beschreibung:
```php
Rückruf Connection::$onBufferDrain
```

Funktioniert wie das [Worker::$onBufferDrain](../worker/on-buffer-drain.md)-Rückruf, mit dem Unterschied, dass es nur für die aktuelle Verbindung wirkt, d. h. Sie können den onBufferDrain-Rückruf für eine bestimmte Verbindung separat einstellen.
