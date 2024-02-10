# onError
## Beschreibung:
```php
Rückruf Connection::$onError
```

Funktioniert genauso wie das [Worker::$onError](../worker/on-error.md) Rückruf, allerdings wirkt es nur für die aktuelle Verbindung, d.h. es kann speziell für einen bestimmten Verbindung onErroe Rückruf eigenständig eingestellt werden.
