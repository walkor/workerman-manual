# onError
## Description:
```php
callback Connection::$onError
```
Il fonctionne de la même manière que le rappel [Worker::$onError](../worker/on-error.md), à la différence qu'il s'applique uniquement à la connexion actuelle, c'est-à-dire qu'il est possible de définir individuellement le rappel onError pour une connexion spécifique.
