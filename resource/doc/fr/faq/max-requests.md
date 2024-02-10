# Comment configurer Workerman pour redémarrer le processus actuel après un certain nombre de requêtes
Pour rendre Workerman plus concis, cette fonction n'est pas directement fournie, mais elle peut être implémentée avec quelques lignes de code.

```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // Traitement métier omis
    if(++$request_count > 10000) {
        // Après 10000 requêtes, arrêtez le processus actuel, le processus principal redémarrera automatiquement un nouveau processus
        Worker::stopAll();
    }
};
```
