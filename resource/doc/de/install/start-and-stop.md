# Starten und Stoppen

Bitte beachten Sie, dass Workerman-Start- und Stop-Befehle alle über die Befehlszeile ausgeführt werden.

Um Workerman zu starten, benötigen Sie zunächst eine Startdatei, in der der Port und das Protokoll definiert sind, auf die der Dienst hören soll. Sie können sich an [den Abschnitt "Einführender Leitfaden - Einfaches Entwicklungsbeispiel"](../getting-started/simple-example.md) halten.

Hier ist ein Beispiel mit [workerman-chat](https://www.workerman.net/workerman-chat), dessen Startdatei start.php lautet.

### Starten

Im Debug-Modus starten

 ```php start.php start```

Als Daemon-Prozess starten

 ```php start.php start -d```

### Stoppen

 ```php start.php stop```

### Neustarten

 ```php start.php restart```

### Sanftes Neustarten

 ```php start.php reload```

### Status anzeigen

 ```php start.php status```

### Verbindungsstatus anzeigen (erfordert Workerman-Version >=3.5.0)

```php start.php connections```

## Unterschiede zwischen Debug- und Daemon-Modus

1. Im Debug-Modus werden Ausgabefunktionen wie echo, var_dump, print usw. direkt in der Konsole ausgegeben.

2. Im Daemon-Modus werden Ausgabefunktionen wie echo, var_dump, print standardmäßig in die Datei /dev/null umgeleitet. Sie können den Dateipfad mit ```Worker::$stdoutFile = '/your/path/file';``` festlegen.

3. Im Debug-Modus wird Workerman beim Schließen der Konsole ebenfalls beendet und beendet.

4. Im Daemon-Modus läuft Workerman nach dem Schließen der Konsole weiterhin normal im Hintergrund.

## Was ist ein sanfter Neustart?

Siehe [Prinzip des sanften Neustarts](../faq/reload-principle.md)
