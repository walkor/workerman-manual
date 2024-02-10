# Erweiterungen installieren
## Hinweis
Im Gegensatz zu Apache+PHP oder Nginx+PHP läuft WorkerMan auf PHP-Befehlszeile [PHP CLI](http://php.net/manual/zh/features.commandline.php) und verwendet ein anderes PHP-Ausführungsprogramm sowie möglicherweise eine unterschiedliche php.ini-Datei. Daher bedeutet das Vorhandensein einer Erweiterung bei der Ausgabe von ```phpinfo()``` auf der Webseite nicht zwangsläufig, dass dieselbe Erweiterung auch in der PHP CLI installiert ist.

## Ermitteln installierter PHP CLI-Erweiterungen
Die Ausführung von ```php -m``` listet die installierten Erweiterungen der PHP CLI auf. Das Ergebnis wäre ähnlich wie unten dargestellt:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## Ermitteln des Speicherorts der php.ini-Datei für PHP CLI
Um die php.ini-Datei für PHP CLI zu finden, führt man ```php --ini``` aus, um den Ort der php.ini-Datei zu ermitteln. Das Ergebnis wäre ähnlich wie unten dargestellt (die Ergebnisse können je nach System variieren):
```shell
~# php --ini
Configuration File (php.ini) Path: /etc/php8/cli
Loaded Configuration File:         /etc/php8/cli/php.ini
Scan for additional .ini files in: /etc/php8/cli/conf.d
Additional .ini files parsed:      /etc/php8/cli/conf.d/apc.ini,
/etc/php8/cli/conf.d/pdo.ini,
/etc/php8/cli/conf.d/pdo_mysql.ini
...
```

# Installation von Erweiterungen für PHP CLI (Installation der memcached-Erweiterung als Beispiel)
## Methode 1: Verwendung von apt oder yum
Wenn PHP mit apt oder yum installiert wurde, können auch Erweiterungen mit apt oder yum installiert werden.

**Installation von PHP-Erweiterungen auf debian/ubuntu-Systemen mit apt (nicht-root Benutzer benötigt sudo-Befehl)**

1. Verwenden von ```apt-cache search```, um das Erweiterungspaket zu suchen
```shell
~# apt-cache search memcached php
php-apc - APC (Alternative PHP Cache) Modul für PHP 5
php5-memcached - memcached-Modul für php5
```

2. Verwenden von ```apt-get install```, um das Erweiterungspaket zu installieren
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**Installation von PHP-Erweiterungen auf centos-Systemen mit yum**

1. Verwenden von ```yum search```, um das Erweiterungspaket zu suchen
```shell
~# yum search memcached php
php-pecl-memcached - memcached-Modul für php5
```

2. Verwenden von ```yum install```, um das Erweiterungspaket zu installieren
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**Anmerkung:**

Die Verwendung von apt oder yum zur Installation von PHP-Erweiterungen führt automatisch zur Konfiguration der php.ini-Datei, was die Installation erleichtert. Ein Nachteil ist, dass einige Erweiterungen in apt oder yum möglicherweise keine entsprechenden Installationspakete haben.

## Methode 2: Verwendung von pecl zur Installation
Verwenden Sie den Befehl ```pecl install```, um die Erweiterung zu installieren.

1. Installation mit ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```

2. Konfiguration der php.ini

Mit ```php --ini``` den Ort der php.ini-Datei finden und die Zeile ```extension=memcached.so``` hinzufügen.

## Methode 3: Kompilieren und Installation aus dem Quellcode (Installation der pcntl-Erweiterung als Beispiel)
1. Verwendung des Befehls ```php -v``` zur Anzeige der aktuellen Version der PHP CLI
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```

2. Herunterladen des PHP-Quellcodes entsprechend der Version

Seite für historische PHP-Versionen: https://php.net/releases/

3. Entpacken des Quellcode-Archivs

Beispiel: Dateiname des heruntergeladenen Archivs ist ```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```

4. Wechseln in das Verzeichnis ext/pcntl im Quellcode
```shell
~# cd php-5.3.29/ext/pcntl/
```

5. Ausführen des Befehls ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

6. Ausführen des Befehls ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```

7. Ausführen des Befehls ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```

8. Ausführen des Befehls ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

9. Konfigurieren der ini-Datei

Mit ```php --ini``` den Ort der php.ini-Datei finden und die Zeile ```extension=pcntl.so``` hinzufügen.

**Anmerkung:**
Diese Methode wird in der Regel verwendet, um PHP-eigene Erweiterungen zu installieren, z. B. posix-Erweiterung und pcntl-Erweiterung. Neben der Kompilierung einer bestimmten Erweiterung mit phpize ist auch die Neu-Kompilierung des gesamten PHP möglich, wobei bei der Kompilierung Argumente für Erweiterungen hinzugefügt werden, z. B. durch Ausführen der folgenden Befehle im Quellcode-Hauptverzeichnis:
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Methode 4: Installation mit phpize
Wenn eine zu installierende Erweiterung im ext-Verzeichnis des PHP-Quellcodes nicht vorhanden ist, kann diese Erweiterung unter https://pecl.php.net gesucht und heruntergeladen werden.

Als Beispiel für die Installation der libevent-Erweiterung (sofern das System die libevent-dev-Bibliothek installiert hat):

1. Herunterladen des libevent-Erweiterungsarchivs (beliebiger Speicherort auf dem aktuellen System)
```shell
~# wget https://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  https://pecl.php.net/get/libevent-0.1.0.tgz
Resolving pecl.php.net... 104.236.228.160
Connecting to pecl.php.net|104.236.228.160|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9806 (9.6K) [application/octet-stream]
Saving to: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s

```

2. Entpacken des Erweiterungsarchivs
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. Wechseln in das Verzeichnis des Quellcodes
```shell
~# cd libevent-0.1.0/
```

4. Ausführen des Befehls ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. Ausführen des Befehls ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6. Ausführen des Befehls ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. Ausführen des Befehls ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8. Konfigurieren der ini-Datei

Mit ```php --ini``` den Ort der php.ini-Datei finden und die Zeile ```extension=libevent.so``` hinzufügen.
