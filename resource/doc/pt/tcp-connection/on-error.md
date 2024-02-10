# onError
## Descrição:
```php
callback Connection::$onError
```

Esta função é equivalente ao callback [Worker::$onError](../worker/on-error.md), com a diferença de que ela afeta apenas a conexão atual. Ou seja, pode ser configurada individualmente para o callback onError de uma determinada conexão.
