# Installationsanleitung
Workerman ist tatsächlich ein PHP-Codepaket. Wenn Ihre PHP-Umgebung bereits installiert ist, müssen Sie nur den Workerman-Quellcode oder das Demo herunterladen, um es auszuführen.

**Composer-Installation:**
```sh
composer require workerman/workerman
```

> **Bemerkung**
> Einige Composer-Proxyspiegel sind unvollständig. Verwenden Sie den obigen Befehl `composer config -g --unset repos.packagist`, um den Proxy zu entfernen.

# Anleitung für Windows-Benutzer (zum Lesen)
Ab der Version Workerman 3.5.3 unterstützt Workerman nun Windows- und Linux-Systeme gleichzeitig.
Windows-Benutzer müssen die Umgebungsvariablen für PHP konfigurieren.

 ` === Ab hier gilt nur für Workerman unter Linux, bitte ignorieren Sie es, wenn Sie ein Windows-Benutzer sind=== `

# Überprüfung der Linux-Systemumgebung
Das Linux-System kann mit dem folgenden Skript die lokale PHP-Umgebung auf die Workerman-Ausführungsanforderungen überprüfen.
 `curl -Ss https://www.workerman.net/check | php`

Wenn das obige Skript "ok" anzeigt, entspricht es den Anforderungen von Workerman, und Sie können direkt das Beispiel von der [offiziellen Website](https://www.workerman.net/) herunterladen und ausführen.

Wenn nicht alles "ok" ist, befolgen Sie die Anweisungen unten, um die fehlenden Erweiterungen zu installieren.

(Hinweis: Das Überprüfungsskript überprüft nicht die Event-Erweiterung. Wenn die Anzahl der gleichzeitigen Verbindungen größer als 1024 ist, müssen Sie die Event-Erweiterung installieren und den [Linux-Kernel optimieren](../appendices/kernel-optimization.md). Die Installationsmethode finden Sie unten.)

# Installation von fehlenden Erweiterungen für vorhandene PHP-Umgebung

## Installation der pcntl- und posix-Erweiterungen:

**CentOS-System**
Wenn PHP über Yum installiert ist, führen Sie folgenden Befehl aus: ```yum install php-process``` zur Installation der pcntl- und posix-Erweiterungen.

Wenn die Installation fehlschlägt oder PHP nicht mit Yum installiert wurde, lesen Sie bitte die Anleitung im Abschnitt [Anhänge - Erweiterungen installieren](../appendices/install-extension.md) Methode drei – Installation aus den Quellen.

**Debian/Ubuntu/Mac OS-Systeme**
Lesen Sie bitte die Anleitung im Abschnitt [Anhänge - Erweiterungen installieren](../appendices/install-extension.md) Methode drei – Installation aus den Quellen.

## Installation der Event-Erweiterung:
Um eine größere Anzahl gleichzeitiger Verbindungen zu unterstützen, ist die Installation der Event-Erweiterung erforderlich. Außerdem muss der [Linux-Kernel optimiert](../appendices/kernel-optimization.md) werden. Die Installationsmethode finden Sie unten:

**CentOS-System**

1. Installieren Sie das libevent-devel-Paket, das für die Event-Erweiterungsabhängigkeit erforderlich ist, über die Befehlszeile: 
```shell
yum install libevent-devel -y
# Wenn die Installation fehlschlägt, versuchen Sie es mit dem folgenden Befehl:
# yum install libevent2-devel -y
```

2. Installieren Sie die Event-Erweiterung über die Befehlszeile (Die Event-Erweiterung erfordert PHP>=5.4): 
```shell
pecl install event
```
Beachten Sie die Aufforderung: ```Include libevent OpenSSL support [yes] :```. Geben Sie "no" ein und drücken Sie die Eingabetaste. Für alle anderen Fragen drücken Sie einfach die Eingabetaste.

3. Führen Sie ```php --ini``` aus, um die php.ini-Datei zu finden und zu öffnen. Fügen Sie am Ende die folgende Konfiguration hinzu:
```shell
extension=event.so
```

**Debian/Ubuntu-System**

1. Installieren Sie das libevent-dev-Paket, das für die Event-Erweiterungsabhängigkeit erforderlich ist, über die Befehlszeile: 
```shell
apt-get install libevent-dev -y
# Wenn die Installation fehlschlägt, versuchen Sie es mit dem folgenden Befehl:
# apt-get install libevent2-dev -y
```

2. Installieren Sie die Event-Erweiterung über die Befehlszeile: 
```shell
pecl install event
```
Beachten Sie die Aufforderung: ```Include libevent OpenSSL support [yes] :```. Geben Sie "no" ein und drücken Sie die Eingabetaste. Für alle anderen Fragen drücken Sie einfach die Eingabetaste.

3. Führen Sie ```php --ini``` aus, um die php.ini-Datei zu finden und zu öffnen. Fügen Sie am Ende die folgende Konfiguration hinzu:
```shell
extension=event.so
```

**Anleitung zur Installation auf Mac OS-Systemen**

Mac-Systeme werden normalerweise als Entwicklungsplattformen verwendet und erfordern daher keine Installation der Event-Erweiterung.

# Neue Systeminstallation (Neue Installation von PHP + Erweiterungen)

## Anleitung zur Installation auf CentOS-Systemen

1. Führen Sie folgenden Befehl aus, um über die Befehlszeile PHP-CLI, pcntl, posix, libevent-Bibliothek und das Git-Programm zu installieren:
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. Installieren Sie die Event-Erweiterung über die Befehlszeile: 
(Beachten Sie: Die Event-Erweiterung erfordert PHP>=5.4)
```shell
pecl install event
```
Beachten Sie die Aufforderung: ```Include libevent OpenSSL support [yes] :```. Geben Sie "no" ein und drücken Sie die Eingabetaste. Für alle anderen Fragen drücken Sie einfach die Eingabetaste.

3. Führen Sie ```php --ini``` aus, um die php.ini-Datei zu finden und zu öffnen. Fügen Sie am Ende die folgende Konfiguration hinzu:
```shell
extension=event.so
```

4. Führen Sie den folgenden Befehl aus, um das Hauptprogramm von Workerman über Github herunterzuladen: 
```shell
git clone https://github.com/walkor/Workerman
```

5. Befolgen Sie den [Einführungshandbuch - Abschnitt über einfache Entwicklungsbeispiele](../getting-started/simple-example.md), um eine Einstiegsdatei zu schreiben und auszuführen. Alternativ können Sie ein fertiges Demo von der [offiziellen Website](https://www.workerman.net/) herunterladen und ausführen.

## Anleitung zur Installation auf Debian/Ubuntu-Systemen

1. Führen Sie folgenden Befehl aus, um über die Befehlszeile PHP-CLI, libevent-Bibliothek und das Git-Programm zu installieren:
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. Installieren Sie die Event-Erweiterung über die Befehlszeile: 
(Beachten Sie: Die Event-Erweiterung erfordert PHP>=5.4)
```shell
pecl install event
```
Beachten Sie die Aufforderung: ```Include libevent OpenSSL support [yes] :```. Geben Sie "no" ein und drücken Sie die Eingabetaste. Für alle anderen Fragen drücken Sie einfach die Eingabetaste.

3. Führen Sie ```php --ini``` aus, um die php.ini-Datei zu finden und zu öffnen. Fügen Sie am Ende die folgende Konfiguration hinzu:
```shell
extension=event.so
```

4. Führen Sie den folgenden Befehl aus, um das Hauptprogramm von Workerman über Github herunterzuladen: 
```shell
git clone https://github.com/walkor/Workerman
```

5. Befolgen Sie den [Einführungshandbuch - Abschnitt über einfache Entwicklungsbeispiele](../getting-started/simple-example.md), um eine Einstiegsdatei zu schreiben und auszuführen. Alternativ können Sie ein fertiges Demo von der [offiziellen Website](https://www.workerman.net/) herunterladen und ausführen.

## Anleitung zur Installation auf Mac OS-Systemen

**Methode 1:** Das Mac-System verfügt standardmäßig über PHP-CLI, es fehlt jedoch möglicherweise die ```pcntl```-Erweiterung.

1. Beachten Sie die Anleitung im Abschnitt [Anhänge - Erweiterungen installieren](../appendices/install-extension.md) Methode drei – Installation aus den Quellen, um die ```pcntl```-Erweiterung zu installieren.

2. Beachten Sie die Anleitung im Abschnitt [Anhänge - Erweiterungen installieren](../appendices/install-extension.md) Methode vier, um die ```event```-Erweiterung mithilfe von phpize zu installieren (Auf Entwicklungsmaschinen kann dieser Schritt übersprungen werden).

3. Laden Sie das Hauptprogramm von Workerman über https://www.workerman.net/download/workermanzip herunter, oder laden Sie ein Beispiel von der [offiziellen Website](https://www.workerman.net/) herunter.

**Methode 2:** Installieren Sie PHP und die entsprechende Erweiterung mithilfe des Befehls ```brew```

1. Führen Sie den folgenden Befehl aus, um das Tool ```brew``` über die Befehlszeile zu installieren (Wenn Sie ```brew``` bereits installiert haben, können Sie diesen Schritt überspringen):
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Führen Sie den folgenden Befehl aus, um ```php``` über die Befehlszeile zu installieren:
```shell
brew install php
```

3. Führen Sie den folgenden Befehl aus, um die ```event```-Erweiterung über die Befehlszeile zu installieren:
```shell
brew install php-event    
```

4. Laden Sie ein Beispiel von der [offiziellen Website](https://www.workerman.net/) herunter.

# Event-Erweiterungserläuterung
Die [Event-Erweiterung](https://php.net/manual/zh/book.event.php) ist nicht unbedingt erforderlich. Wenn Ihr Geschäft jedoch mehr als 1000 gleichzeitige Verbindungen erfordert, wird die Installation von Event empfohlen, da sie eine riesige Anzahl von gleichzeitigen Verbindungen unterstützen kann. Wenn Ihr Geschäft weniger als 1000 gleichzeitige Verbindungen erfordert, ist die Installation nicht erforderlich.

## Häufige Probleme
1. Wenn folgende Fehlermeldung angezeigt wird: `checking for include/event2/event.h... not found`, versuchen Sie zuerst, das libevent-dev(el)-Paket zu deinstallieren und libevent2-dev(el) neu zu installieren.
  - CentOS-System: `yum remove libevent-devel && yum install libevent2-devel`
  - Debian/Ubuntu-System: `apt-get remove libevent-dev && apt-get install libevent2-dev`

2. Wenn folgende Fehlermeldung angezeigt wird: `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
   Ändern Sie die Reihenfolge des Ladens von event.so und socket.so, dh. fügen Sie in der php.ini-Datei `extension=socket.so` vor `extension=event.so` ein, damit die Socket-Erweiterung zuerst geladen wird.
