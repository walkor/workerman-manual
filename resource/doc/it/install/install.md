# Istruzioni per l'installazione
Workerman è essenzialmente un pacchetto di codice PHP e se hai già PHP installato sul tuo ambiente, è sufficiente scaricare il codice sorgente di Workerman o la demo per eseguirlo.

**Installazione tramite Composer:**
```sh
composer require workerman/workerman
```

> **Nota**
> Alcuni mirror di Composer potrebbero non essere completi, per utilizzare il comando sopra, esegui `composer config -g --unset repos.packagist` per rimuovere il proxy.

# Utenti Windows (essenziale)
A partire dalla versione 3.5.3 di Workerman, il sistema è compatibile sia con Windows che con Linux. Gli utenti Windows devono configurare le variabili di ambiente di PHP.

` === Questa pagina si applica solo all'ambiente di sviluppo Linux di Workerman, ignorare se si è utenti Windows === `

# Verifica dell'ambiente di sistema Linux
Per testare se l'ambiente PHP locale soddisfa i requisiti di esecuzione di Workerman su Linux, esegui lo script seguente:
`curl -Ss https://www.workerman.net/check | php`

Se tutte le verifiche dello script restituiscono "ok", significa che i requisiti di Workerman sono soddisfatti e puoi scaricare l'esempio dal [sito ufficiale](https://www.workerman.net/) per l'esecuzione.

Se non tutte le verifiche restituiscono "ok", segui la documentazione di seguito per installare le estensioni mancanti.

(Nota: lo script di verifica non include la verifica dell'estensione "event". Se il numero di connessioni concorrenti supera 1024, è necessario installare l'estensione "event" e ottimizzare il kernel di Linux, per i metodi di installazione delle estensioni, fare riferimento alle istruzioni seguenti)

# Installazione delle estensioni mancanti su un ambiente PHP esistente

## Installazione dell'estensione pcntl e posix:

**Sistema CentOS**
Se PHP è stato installato tramite yum, esegui il seguente comando:
```sh
yum install php-process
```
per installare le estensioni pcntl e posix.

Se l'installazione fallisce o PHP non è stato installato tramite yum, segui il metodo tre della documentazione [Appendice - Installazione Estensioni](../appendices/install-extension.md) per l'installazione tramite compilazione da codice sorgente.

**Sistemi Debian/Ubuntu/Mac OS**
Segui il metodo tre della documentazione [Appendice - Installazione Estensioni](../appendices/install-extension.md) per l'installazione tramite compilazione da codice sorgente.

## Installazione dell'estensione "event":
Per supportare un numero maggiore di connessioni concorrenti, è necessario installare l'estensione "event" e ottimizzare il kernel di Linux. I passaggi per l'installazione sono i seguenti:

**Sistema CentOS**

1. Installa il pacchetto libevent-devel richiesto dall'estensione "event" con il comando:
```shell
yum install libevent-devel -y
# Se non riesci a installare, prova con il seguente comando
# yum install libevent2-devel -y
```

2. Installa l'estensione "event" con il seguente comando:
(L'estensione "event" richiede PHP>=5.4)
```shell
pecl install event
```
Quando richiesto di includere il supporto a OpenSSL per libevent, rispondi "no" e premi Invio.

3. Esegui il comando `php --ini` per individuare e aprire il file php.ini, quindi aggiungi la seguente configurazione alla fine del file:
```shell
extension=event.so
```

**Sistemi Debian/Ubuntu**

1. Installa il pacchetto libevent-dev richiesto dall'estensione "event" con il comando:
```shell
apt-get install libevent-dev -y
# Se non riesci a installare, prova con il seguente comando
# apt-get install libevent2-dev -y
```

2. Installa l'estensione "event" con il seguente comando:
```shell
pecl install event
```
Quando richiesto di includere il supporto a OpenSSL per libevent, rispondi "no" e premi Invio.

3. Esegui il comando `php --ini` per individuare e aprire il file php.ini, quindi aggiungi la seguente configurazione alla fine del file:
```shell
extension=event.so
```

**Guida all'installazione su Mac OS**

Di solito, su Mac viene utilizzato come ambiente di sviluppo e l'estensione "event" non è necessaria.

# Installazione del sistema completo (Installazione completa di PHP + estensioni)

## Guida all'installazione su sistemi CentOS

1. Esegui il seguente comando da terminale (questo passaggio include l'installazione del programma principale php-cli e delle estensioni pcntl, posix, libevent e git):
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. Installa l'estensione "event" con il seguente comando:
(L'estensione "event" richiede PHP>=5.4)
```shell
pecl install event
```
Quando richiesto di includere il supporto a OpenSSL per libevent, rispondi "no" e premi Invio.

3. Esegui il comando `php --ini` per individuare e aprire il file php.ini, quindi aggiungi la seguente configurazione alla fine del file:
```shell
extension=event.so
```

4. Esegui il comando da terminale (questo passaggio scarica il programma principale di Workerman da GitHub):
```shell
git clone https://github.com/walkor/Workerman
```

5. Segui la sezione [Guida introduttiva - Esempi di sviluppo semplici](../getting-started/simple-example.md) per scrivere il file di ingresso e eseguirlo.
Oppure scarica e esegui l'esempio dal [sito ufficiale](https://www.workerman.net/).

## Guida all'installazione su sistemi Debian/Ubuntu

1. Esegui il seguente comando da terminale (questo passaggio include l'installazione del programma principale php-cli, libevent e git):
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. Installa l'estensione "event" con il seguente comando:
(L'estensione "event" richiede PHP>=5.4)
```shell
pecl install event
```
Quando richiesto di includere il supporto a OpenSSL per libevent, rispondi "no" e premi Invio.

3. Esegui il comando `php --ini` per individuare e aprire il file php.ini, quindi aggiungi la seguente configurazione alla fine del file:
```shell
extension=event.so
```

4. Esegui il comando da terminale (questo passaggio scarica il programma principale di Workerman da GitHub):
```shell
git clone https://github.com/walkor/Workerman
```

5. Segui la sezione [Guida introduttiva - Esempi di sviluppo semplici](../getting-started/simple-example.md) per scrivere il file di ingresso e eseguirlo.
Oppure scarica e esegui l'esempio dal [sito ufficiale](https://www.workerman.net/).

## Guida all'installazione su Mac OS

**Metodo 1:** Il sistema Mac dispone già di PHP Cli, ma potrebbe mancare l'estensione `pcntl`.

1. Segui la documentazione [Appendice - Installazione Estensioni](../appendices/install-extension.md) e compila e installa l'estensione `pcntl`.

2. Segui la documentazione [Appendice - Installazione Estensioni](../appendices/install-extension.md) e usa phpize per installare l'estensione `event` (questo passaggio può essere omesso se si tratta di un ambiente di sviluppo).

3. Scarica il programma principale di Workerman da https://www.workerman.net/download/workermanzip o dal [sito ufficiale](https://www.workerman.net/).

**Metodo 2:** Installa PHP e le relative estensioni tramite il comando `brew`.

1. Esegui i seguenti comandi da terminale per installare lo strumento `brew` (se `brew` è già installato, questo passaggio può essere saltato):
```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Esegui il seguente comando da terminale per installare PHP:
```sh
brew install php
```

3. Esegui il seguente comando da terminale per installare l'estensione `event`:
```sh
brew install php-event
```

4. Scarica e esegui l'esempio dal [sito ufficiale](https://www.workerman.net/).

# Spiegazione dell'estensione "Event"
L'estensione [Event](https://php.net/manual/zh/book.event.php) non è obbligatoria, ma è consigliata se l'applicazione richiede un elevato numero di connessioni concorrenti (superiore a 1000), in quanto supporta un grande numero di connessioni concorrenti. Se il numero di connessioni concorrenti è inferiore a 1000, l'installazione non è necessaria.

## Domande frequenti
1. Se compare l'errore `checking for include/event2/event.h... not found`, prova a rimuovere la libreria libevent-devel e a installare libevent2-devel.
   CentOS: yum remove libevent-devel && yum install libevent2-devel
   Debian/Ubuntu: apt-get remove libevent-dev && apt-get install libevent2-dev

2. Se compare l'errore `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
   Modifica l'ordine di caricamento di event.so e socket.so, cioè nel file php.ini, posiziona `extension=socket.so` prima di `extension=event.so` in modo che l'estensione del socket venga caricata per prima.
