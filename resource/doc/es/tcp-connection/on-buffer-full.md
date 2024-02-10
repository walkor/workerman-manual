# onBufferFull
## Descripción:
```php
callback Connection::$onBufferFull
```

Funciona igual que el callback [Worker::$onBufferFull](../worker/on-buffer-full.md), la diferencia es que solo afecta a la conexión actual, es decir, se puede configurar individualmente el callback onBufferFull para una conexión específica.
