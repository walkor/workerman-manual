# Installazione dell'estensione
## Nota
Diversamente dalla modalità di esecuzione di Apache+PHP o Nginx+PHP, WorkerMan si basa sull'esecuzione della riga di comando PHP [PHP CLI](http://php.net/manual/zh/features.commandline.php), utilizzando un diverso programma eseguibile PHP e potenzialmente un diverso file php.ini. Pertanto, stampare ```phpinfo()``` su una pagina web per vedere se un'estensione è installata non significa che l'ambiente PHP CLI abbia anche installato l'estensione corrispondente.

## Come verificare quali estensioni sono installate in PHP CLI
Eseguire il comando ```php -m``` per elencare le estensioni installate in PHP CLI, il risultato sarà simile al seguente:
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## Come trovare la posizione del file php.ini di PHP CLI
Quando si installa un'estensione, potrebbe essere necessario configurare manualmente il file php.ini per aggiungere l'estensione. È quindi necessario confermare la posizione del file php.ini di PHP CLI. Si può eseguire il comando ```php --ini``` per cercare la posizione del file ini di PHP CLI, il risultato sarà simile al seguente (i risultati possono variare tra i vari sistemi):
```shell
~# php --ini
Percorso del file di configurazione (php.ini): /etc/php8/cli
File di configurazione caricato: /etc/php8/cli/php.ini
Scansione di file .ini aggiuntivi in: /etc/php8/cli/conf.d
File .ini aggiuntivi analizzati: /etc/php8/cli/conf.d/apc.ini,
/etc/php8/cli/conf.d/pdo.ini,
/etc/php8/cli/conf.d/pdo_mysql.ini
...
```

# Installazione dell'estensione PHP CLI (installazione dell'estensione memcached come esempio)
## Metodo 1: Installazione mediante apt o yum
Se PHP è stato installato tramite apt o yum, le estensioni possono essere installate anche tramite apt o yum.

**Metodo di installazione dell'estensione php su sistemi debian/ubuntu tramite apt (gli utenti non root devono aggiungere il comando sudo)**

1. Utilizzare il comando ```apt-cache search``` per cercare il pacchetto dell'estensione
```shell
~# apt-cache search memcached php
php-apc - Modulo APC (Alternative PHP Cache) per PHP 5
php5-memcached - modulo memcached per php5
```
2. Utilizzare il comando ```apt-get install``` per installare il pacchetto dell'estensione
```shell
~# apt-get install -y php5-memcached
Lettura della lista dei pacchetti... Fatto
Lettura dell'elenco dei pacchetti... Fatto
...
```

**Metodo di installazione dell'estensione php su sistemi centos tramite yum**

1. Utilizzare il comando ```yum search``` per cercare il pacchetto dell'estensione
```shell
~# yum search memcached php
php-pecl-memcached - modulo memcached per php5
```
2. Utilizzare il comando ```yum install``` per installare il pacchetto dell'estensione
```shell
~# yum install -y php-pecl-memcached
Lettura della lista dei pacchetti... Fatto
Lettura dell'elenco dei pacchetti... Fatto
...
```
**Nota:**
L'installazione delle estensioni PHP tramite apt o yum include automaticamente la configurazione del file php.ini, rendendo le estensioni disponibili senza ulteriori passaggi. Tuttavia, alcune estensioni potrebbero non essere disponibili tramite apt o yum.

## Metodo 2: Installazione tramite pecl
Utilizzare il comando ```pecl install``` per installare l'estensione

1. Installazione con ```pecl install```
```shell
~# pecl install memcached
Download di memcached-2.2.0.tgz ...
Inizio del download di memcached-2.2.0.tgz (70,449 byte)
....
```
2. Configurare php.ini

Usando il comando ```php --ini``` per trovare la posizione del file php.ini, quindi aggiungere ```extension=memcached.so``` al file.

# Metodo 3: Installazione tramite compilazione del codice sorgente (generalmente per l'installazione di estensioni fornite con PHP, esempio di installazione dell'estensione pcntl)
1. Utilizzare il comando ```php -v``` per verificare la versione corrente della riga di comando PHP CLI
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Scaricare il codice sorgente di PHP in base alla versione
  
   Pagina di download delle versioni storiche di PHP: https://php.net/releases/

3. Estrarre l'archivio compresso del codice sorgente

   Ad esempio, se il nome del file compresso è ```php-5.3.29.tar.gz```
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. Accedere alla directory ext/pcntl del codice sorgente
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. Eseguire il comando ```phpize```
```shell
~# phpize
Configurazione di:
Versione API di PHP:         20090626
Numero API del modulo Zend: 20090626
Numero API dell'estensione Zend: 220090626
```
6. Eseguire il comando ```configure```
```shell
~# ./configure
controllo di grep che gestisce righe lunghe e -e... /bin/grep
controllo di egrep... /bin/grep -E
...
```
7. Eseguire il comando ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. Eseguire il comando ```make install```
```shell
~# make install
Installazione di estensioni condivise:     /usr/lib/php5/20090626/
```
9. Configurare il file ini

Utilizzare il comando ```php --ini``` per trovare la posizione del file php.ini, quindi aggiungere ```extension=pcntl.so``` al file.

**Nota:**
Questo metodo viene generalmente utilizzato per installare estensioni fornite con PHP, come ad esempio l'estensione posix e l'estensione pcntl. Oltre a compilare un'estensione con phpize, è possibile ricompilare l'intero PHP aggiungendo l'estensione come parametro durante la compilazione, ad esempio eseguendo il seguente comando nella directory radice del codice sorgente
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Metodo 4: Installazione tramite phpize
Se l'estensione da installare non è presente nella directory ext del codice sorgente di PHP, è necessario scaricarla da https://pecl.php.net e installarla.

Prendiamo come esempio l'installazione dell'estensione libevent (supponendo che il sistema abbia installato la libreria libevent-dev)

1. Scaricare il file compresso dell'estensione libevent (in qualsiasi directory del sistema)
```shell
~# wget https://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  https://pecl.php.net/get/libevent-0.1.0.tgz
Risoluzione di pecl.php.net... 104.236.228.160
Connessione a pecl.php.net|104.236.228.160|:80... connesso.
Richiesta HTTP inviata, aspetto risposta... 200 OK
Lunghezza: 9806 (9.6K) [application/octet-stream]
Salvataggio in: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s
```

2. Estrarre il file compresso dell'estensione
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. Accedere alla cartella sorgente
```shell
~# cd libevent-0.1.0/
```

4. Eseguire il comando ```phpize```
```shell
~# phpize
Configurazione di:
Versione API di PHP:         20090626
Numero API del modulo Zend: 20090626
Numero API dell'estensione Zend: 220090626
```

5. Eseguire il comando ```configure```
```shell
~# ./configure
controllo di grep che gestisce righe lunghe e -e... /bin/grep
controllo di egrep... /bin/grep -E
controllo di a sed che non tronca l'output... /bin/sed
controllo di cc... cc
controllo che il compilatore C funzioni... sì
...
```

6. Eseguire il comando ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. Eseguire il comando ```make install```
```shell
~# make install
Installazione di estensioni condivise:     /usr/lib/php5/20090626/
```

8. Configurare il file ini

Utilizzare il comando ```php --ini``` per trovare la posizione del file php.ini, quindi aggiungere ```extension=libevent.so``` al file.
