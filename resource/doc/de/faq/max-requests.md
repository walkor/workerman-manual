# Wie man Workerman so einstellt, dass es nach einer bestimmten Anzahl von Anfragen den aktuellen Prozess neu startet
Um Workerman schlanker zu machen, wird diese Einstellung nicht direkt bereitgestellt, aber Sie können diese Funktion mit nur wenigen Zeilen Code implementieren.
```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // Business-Logik ausgelassen
    if(++$request_count > 10000) {
        // Wenn die Anzahl der Anfragen 10000 erreicht hat, wird der aktuelle Prozess beendet, und der Hauptprozess startet automatisch einen neuen Prozess neu
        Worker::stopAll();
    }
};
```
