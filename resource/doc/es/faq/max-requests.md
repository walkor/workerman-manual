# Cómo configurar Workerman para reiniciar el proceso actual después de cierto número de solicitudes
Para hacer que Workerman sea más conciso, no proporciona directamente esta configuración, pero se puede lograr esta funcionalidad con unas pocas líneas de código.

```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // Operaciones comerciales omitidas
    if (++$request_count > 10000) {
        // Después de que se alcancen 10000 solicitudes, se detiene el proceso actual y el proceso principal automáticamente reiniciará un nuevo proceso
        Worker::stopAll();
    }
};
```
