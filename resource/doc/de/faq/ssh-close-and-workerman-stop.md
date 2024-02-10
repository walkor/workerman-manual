# Beenden des Terminals führt zum Beenden des Dienstes
**Frage:**

Warum beendet sich Workerman, wenn ich das Terminal schließe?

**Antwort:**

Workerman hat zwei Startmodi, den Debug-Modus und den Daemon-Modus. 

Wenn Sie ```php xxx.php start``` ausführen, wird der Debug-Modus gestartet, der zum Entwickeln und Debuggen von Problemen verwendet wird. Wenn das Terminal geschlossen wird, wird auch Workerman beendet.

Wenn Sie ```php xxx.php start -d``` ausführen, starten Sie den Daemon-Modus, bei dem das Schließen des Terminals Workerman nicht beeinträchtigt.

Wenn Sie möchten, dass Workerman nicht vom Terminal beeinflusst wird, können Sie den Daemon-Modus zum Starten verwenden.
