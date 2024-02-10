# Systemanforderungen

## Nutzer von Windows
Ab der Version 3.5.3 unterstützt Workerman sowohl das Linux- als auch das Windows-Betriebssystem.

1. Sie benötigen PHP ab Version 5.4 und müssen die Umgebungsvariablen für PHP konfiguriert haben.
2. Die Windows-Version von Workerman hängt nicht von Erweiterungen ab.
3. Siehe Installations- und Nutzungsbeschränkungen [hier](https://www.workerman.net/windows).
4. Aufgrund der verschiedenen Nutzungsbeschränkungen von Workerman unter Windows wird empfohlen, das System nur in Entwicklungsumgebungen zu verwenden, während für den produktiven Einsatz ein Linux-System empfohlen wird.

``` ==== Die folgenden Informationen gelten nur für Linux-Benutzer. Windows-Benutzer bitte ignorieren. ====```


## Linux-Benutzer (einschließlich Mac OS)
Linux-Benutzer können nur die Linux-Version von Workerman verwenden.

1. Installieren Sie PHP ab Version 5.4 und haben Sie die Erweiterungen pcntl und posix installiert.
2. Es wird empfohlen, die Event-Erweiterung zu installieren, jedoch ist dies nicht zwingend erforderlich (Beachten Sie, dass die Event-Erweiterung mindestens PHP-Version 5.4 erfordert).

### Skript zur Überprüfung der Linux-Umgebung
Linux-Benutzer können das folgende Skript ausführen, um zu überprüfen, ob ihre lokale Umgebung den Anforderungen von WorkerMan entspricht.

```curl -Ss https://www.workerman.net/check | php```

Wenn das Skript ausschließlich "ok" anzeigt, erfüllt die WorkerMan-Ausführungsumgebung die Anforderungen.

(Hinweis: Das Prüfskript überprüft nicht die Event-Erweiterung. Wenn die Anzahl der gleichzeitigen Verbindungen größer als 1024 ist, wird empfohlen, die Event-Erweiterung zu installieren. Siehe nächster Abschnitt für Informationen zur Installation.)

## Detaillierte Erläuterung

### Über PHP-CLI

WorkerMan wird im [PHP-Befehlszeilenmodus (PHP-CLI)](https://php.net/manual/zh/features.commandline.php) ausgeführt. PHP-CLI ist ein eigenständiges ausführbares Programm und nicht in Konflikt mit PHP-FPM oder dem Apache-Modul PHP-MOD und hängt nicht voneinander ab.

### Über die von WorkerMan abhängigen Erweiterungen

1. [pcntl-Erweiterung](https://cn2.php.net/manual/zh/book.pcntl.php): Die pcntl-Erweiterung ist eine wichtige Erweiterung für die Prozesskontrolle von PHP in der Linux-Umgebung. WorkerMan nutzt Funktionen wie [Prozesserstellung](https://cn2.php.net/manual/zh/function.pcntl-fork.php), [Signalsteuerung](https://cn2.php.net/manual/zh/function.pcntl-signal.php), [Timer](https://cn2.php.net/manual/zh/function.pcntl-alarm.php) und [Überwachung des Prozessstatus](https://cn2.php.net/manual/zh/function.pcntl-waitpid.php). Diese Erweiterung wird nicht auf der Windows-Plattform unterstützt.

2. [posix-Erweiterung](https://cn2.php.net/manual/zh/book.posix.php): Die posix-Erweiterung ermöglicht es PHP in der Linux-Umgebung, auf die vom [POSIX-Standard](https://baike.baidu.com/view/209573.htm) bereitgestellten Schnittstellen zuzugreifen. WorkerMan verwendet hauptsächlich Schnittstellen, um die Funktionen der Daemonisierung und der Benutzergruppenkontrolle zu implementieren. Diese Erweiterung wird nicht auf der Windows-Plattform unterstützt.

3. [Event-Erweiterung](https://php.net/manual/zh/book.event.php) oder [libevent-Erweiterung](https://cn2.php.net/manual/en/book.libevent.php): Die Event-Erweiterung ermöglicht es PHP, fortgeschrittene Ereignisbehandlungsmechanismen wie [Epoll](https://baike.baidu.com/view/1385104.htm) und Kqueue zu nutzen, um die CPU-Auslastung von WorkerMan bei gleichzeitigen Verbindungen erheblich zu verringern. Sie ist besonders wichtig für Anwendungen mit hoher gleichzeitiger Verbindung zur Langzeitverbindung. Die libevent-Erweiterung (bzw. die Event-Erweiterung) ist optional. Wenn sie nicht installiert ist, wird standardmäßig der nativen Select-Ereignisbehandlungsmechanismus von PHP verwendet.

## Installation der Erweiterungen

Siehe Abschnitt [Erweiterungen installieren](../appendices/install-extension.md)
