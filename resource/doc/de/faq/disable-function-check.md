# Deaktivierung der Funktionsüberprüfung

Überprüfen Sie mit diesem Skript, ob bestimmte Funktionen deaktiviert sind. Führen Sie den Befehl ```curl -Ss https://www.workerman.net/check | php``` im Befehlszeilenmodus aus.

Wenn die Meldung ```Function Funktionenname may be disabled. Please check disable_functions in php.ini``` erscheint, bedeutet das, dass die von Workerman abhängigen Funktionen deaktiviert sind. Um Workerman normal verwenden zu können, müssen Sie die Deaktivierung in der php.ini aufheben.
Sie können die Deaktivierung auf eine der folgenden Arten aufheben.

## Methode 1: Deaktivierung über Skript

Führen Sie das Skript `curl -Ss https://www.workerman.net/fix | php` aus, um die Deaktivierung aufzuheben.

## Methode 2: Manuelle Aufhebung

**Die Schritte sind wie folgt:**

1. Führen Sie `php --ini` aus, um den Speicherort der verwendeten php.ini-Datei für die PHP-CLI zu finden.

2. Öffnen Sie die php.ini-Datei und suchen Sie nach dem Eintrag `disable_functions`, um die Deaktivierung der entsprechenden Funktionen aufzuheben.

**Abhängige Funktionen**
Um Workerman verwenden zu können, müssen folgende Funktionen aktiviert werden:
``` 
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
