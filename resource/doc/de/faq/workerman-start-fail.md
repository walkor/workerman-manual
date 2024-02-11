# Workerman konnte nicht gestartet werden

## Phänomen 1
Beim Starten tritt ein Fehler auf, der ähnlich ist wie folgt:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx
```
**Schlüsselwort**: ```Address already in use```

**Grund**: Der Port ist belegt und der Startvorgang kann nicht durchgeführt werden.

#### Lösung 1
Mit dem Befehl ```netstat -anp | grep Portnummer``` kann das Programm identifiziert werden, das den Port belegt. Anschließend kann das entsprechende Programm beendet werden, um den Port freizugeben.

#### Lösung 2
Falls es nicht möglich ist, das Programm zu beenden, das den Port belegt, kann man das Problem durch das Wechseln des Workerman-Ports lösen.

#### Lösung 3
Wenn der von Workerman belegte Port nicht durch den Befehl "stop" gestoppt werden kann (normalerweise aufgrund des fehlenden PID-Datei oder weil der Hauptprozess vom Entwickler beendet wurde), können die folgenden beiden Befehle ausgeführt werden, um den Workerman-Prozess zu beenden:
```bash
killall php
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

#### Lösung 4
Wenn tatsächlich kein Programm den Port belegt, könnte es sein, dass der Entwickler in Workerman zwei oder mehrere Listener mit dem gleichen Port konfiguriert hat. In diesem Fall muss der Entwickler das Startskript überprüfen, um sicherzustellen, dass dieselben Ports verwendet werden.

#### Lösung 5
Überprüfen, ob das Programm "reusePort" aktiviert ist, und deaktivieren, um es zu testen.

## Phänomen 2
Beim Starten tritt ein Fehler auf, der ähnlich ist wie folgt:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
oder
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (在其上下文中，该请求的地址无效) in ...workerman/Worker.php on line xxxx
```
**Schlüsselwort**: `Cannot assign requested address` oder `该请求的地址无效`

**Grund für das Scheitern**: 
Falsche IP-Parameter im Startskript, die nicht zur lokalen IP-Adresse gehören. Es sollte die lokale IP-Adresse angegeben werden oder "0.0.0.0" (um alle lokalen IPs zu überwachen).

**Hinweis**: Auf einem Linux-System können Sie mit dem Befehl ```ifconfig``` alle IP-Adressen Ihres lokalen Netzwerks anzeigen. Wenn Sie ein Cloud-Server (z. B. Alibaba Cloud/Tencent Cloud) Benutzer sind, beachten Sie, dass Ihre öffentliche IP-Adresse möglicherweise eine Proxy-IP-Adresse ist (z. B. das dedizierte Netzwerk von Alibaba Cloud), und die öffentliche IP-Adresse gehört nicht zum aktuellen Server. Daher ist es nicht möglich, die öffentliche IP-Adresse zu überwachen. Dennoch können Sie weiterhin "0.0.0.0" verwenden, um den Server zu binden.

## Phänomen 3
```php
Waring stream_socket_server has been disabled for security reasons in ...
```
**Grund für das Scheitern**: 
Die Funktion stream_socket_server ist in der php.ini-Datei deaktiviert.

**Lösung**:

1. Führen Sie ```php --ini``` aus, um die php.ini-Datei zu finden.
2. Öffnen Sie die php.ini-Datei und entfernen Sie den Eintrag für die Deaktivierung von stream_socket_server in der disable_functions-Sektion.

## Phänomen 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Grund für das Scheitern**: 
Unter Linux benötigen Sie Root-Berechtigungen, um Ports unter 1024 zu überwachen.

**Lösung**:

Verwenden Sie Ports über 1024 oder starten Sie den Dienst als Root-Benutzer.
