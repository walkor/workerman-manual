# Requisiti di sistema

## Utenti Windows
A partire dalla versione 3.5.3, workerman supporta sia i sistemi Linux che Windows.

1. È necessario avere PHP >= 5.4 e configurare correttamente le variabili di ambiente di PHP.

2. La versione Windows di Workerman non dipende da nessuna estensione.

3. Per informazioni sull'installazione e sulle limitazioni d'uso, fare riferimento a [**questo link**](https://www.workerman.net/windows).

4. Poiché Workerman ha molte limitazioni d'uso su Windows, si consiglia di utilizzare il sistema Linux per l'ambiente di produzione e di utilizzare Windows solo per l'ambiente di sviluppo.

``` ====Le informazioni seguenti si applicano solo agli utenti Linux, gli utenti Windows possono ignorarle. ====```

## Utenti Linux (incluso Mac OS)
Gli utenti Linux possono utilizzare solo la versione di Workerman per Linux.

1. È necessario installare PHP >= 5.4 e avere le estensioni pcntl e posix installate.

2. È consigliabile installare l'estensione event, ma non è obbligatoria (si noti che l'estensione event richiede PHP >= 5.4).

### Script di controllo dell'ambiente Linux
Gli utenti Linux possono eseguire lo script seguente per verificare se l'ambiente locale soddisfa i requisiti di WorkerMan

```curl -Ss https://www.workerman.net/check | php```

Se lo script restituisce solo "ok", significa che l'ambiente di esecuzione di WorkerMan è soddisfatto

(Nota: lo script di controllo non verifica l'estensione event. Se il numero di connessioni simultanee supera 1024, si consiglia di installare l'estensione event. Per istruzioni sull'installazione, fare riferimento alla sezione successiva)

## Informazioni dettagliate

### Riguardo a PHP-CLI

WorkerMan viene eseguito in modalità [PHP Command Line Interface (PHP-CLI)](https://php.net/manual/it/features.commandline.php). PHP-CLI è un programma eseguibile indipendente da PHP-FPM o da MOD-PHP di Apache, quindi non entra in conflitto né dipende da essi.

### Estensioni richieste da WorkerMan

1. [Estensione pcntl](https://www.php.net/manual/it/book.pcntl.php)

L'estensione pcntl è essenziale per il controllo dei processi in ambiente Linux. WorkerMan utilizza funzionalità come la [creazione di processi](https://www.php.net/manual/it/function.pcntl-fork.php), il [controllo dei segnali](https://www.php.net/manual/it/function.pcntl-signal.php), il [timer](https://www.php.net/manual/it/function.pcntl-alarm.php) e il [monitoraggio dello stato dei processi](https://www.php.net/manual/it/function.pcntl-waitpid.php). Questa estensione non è supportata su piattaforma Windows.

2. [Estensione posix](https://www.php.net/manual/it/book.posix.php)

L'estensione posix consente a PHP di chiamare le interfacce fornite dal sistema operativo in conformità allo standard [POSIX](https://it.wikipedia.org/wiki/POSIX). WorkerMan utilizza principalmente queste interfacce per implementare funzionalità come i processi in esecuzione in background e il controllo dei gruppi utente. Anche questa estensione non è supportata su piattaforma Windows.

3. [Estensione Event](https://www.php.net/manual/it/book.event.php) o [Estensione Libevent](https://www.php.net/manual/en/book.libevent.php)

L'estensione Event permette a PHP di utilizzare avanzati meccanismi di gestione degli eventi come [Epoll](https://it.wikipedia.org/wiki/Epoll), Kqueue, consentendo di migliorare notevolmente l'utilizzo della CPU da parte di WorkerMan in caso di elevate connessioni simultanee. È particolarmente importante in applicazioni con numerose connessioni attive. Se non installata, WorkerMan utilizzerà di default il meccanismo di gestione eventi nativo di PHP, Select.

## Come installare le estensioni

Consultare la sezione [Installazione delle estensioni](../appendices/install-extension.md)
