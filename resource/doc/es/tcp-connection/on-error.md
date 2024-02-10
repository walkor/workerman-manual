# onError
## Descripción:
```php
callback Connection::$onError
```

Tiene el mismo efecto que la devolución de llamada [Worker::$onError](../worker/on-error.md), la diferencia es que solo se aplica a la conexión actual, es decir, se puede configurar individualmente la devolución de llamada onError para una conexión específica.
