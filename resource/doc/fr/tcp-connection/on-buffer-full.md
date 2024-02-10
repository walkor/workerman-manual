# onBufferFull
## Description:
```php
callback Connection::$onBufferFull
```

Cela fonctionne de la même manière que le rappel [Worker::$onBufferFull](../worker/on-buffer-full.md), à la différence qu'il s'applique uniquement à la connexion actuelle. Autrement dit, il est possible de définir individuellement le rappel onBufferFull pour une connexion spécifique.
