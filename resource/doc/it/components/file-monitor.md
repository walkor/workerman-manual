# Componente di monitoraggio dei file

**Background:**

Workerman è in esecuzione in memoria, il che può evitare la lettura ripetitiva del disco e l'interpretazione ripetuta del PHP, al fine di ottenere le massime prestazioni. Quindi, dopo aver apportato modifiche al codice aziendale, è necessario eseguire manualmente il reload o il restart per renderle effettive.

Allo stesso tempo, Workerman fornisce un servizio di monitoraggio dell'aggiornamento dei file che, quando rileva un aggiornamento del file, esegue automaticamente il reload, ricaricando i file PHP. Gli sviluppatori possono integrarlo nel progetto e avviarlo insieme al progetto.

**Indirizzo di download del servizio di monitoraggio dei file:**

1. Versione senza dipendenze: https://github.com/walkor/workerman-filemonitor

2. Versione dipendente da inotify: https://github.com/walkor/workerman-filemonitor-inotify (richiede l'installazione dell'estensione [inotify](https://php.net/manual/zh/book.inotify.php))

**Differenze tra le due versioni:**

La versione all'indirizzo 1 utilizza il metodo del polling ogni secondo per verificare se il file è stato aggiornato.

La versione all'indirizzo 2 sfrutta il meccanismo del kernel Linux [inotify](https://baike.baidu.com/view/2645027.htm), il quale notifica automaticamente a Workerman quando un file viene aggiornato.

In generale, è sufficiente utilizzare la prima versione senza dipendenze.

**Modalità di utilizzo:**

Copia la directory Applications/FileMonitor nella directory Applications del tuo progetto.

Se il tuo progetto non ha una directory Applications, copia il file Applications/FileMonitor/start.php in qualsiasi posizione del tuo progetto e richiamalo nello script di avvio.

Il componente di monitoraggio monitora di default la directory Applications. Se necessario, puoi modificare il valore della variabile `$monitor_dir` nel file Applications/FileMonitor/start.php. Si consiglia di utilizzare un percorso assoluto per il valore di `$monitor_dir`.

**Nota:**

* Il sistema Windows non supporta il reload e quindi non può utilizzare questo servizio di monitoraggio.
* Funziona solo in modalità debug, non verrà eseguito il monitoraggio dei file in modalità daemon (il motivo per cui la modalità daemon non è supportata è spiegato di seguito).
* Solo i file caricati dopo l'esecuzione di Worker::runAll possono essere aggiornati in modo dinamico, ovvero solo i file caricati nei callback onXXX possono essere aggiornati in modo dinamico.

**Perché non supportare la modalità daemon?**

La modalità daemon è generalmente la modalità in cui viene eseguito l'ambiente di produzione online. Durante il rilascio in ambiente di produzione, di solito vengono distribuiti più file contemporaneamente e potrebbero esserci dipendenze tra i file. Poiché la sincronizzazione di più file su disco richiede del tempo, potrebbe verificarsi una situazione in cui i file sul disco non sono completi in un determinato momento. In questo caso, se il monitoraggio rileva un aggiornamento dei file ed esegue un reload, potrebbero verificarsi errori fatali dovuti alla mancanza di file.

Inoltre, in ambiente di produzione, a volte è necessario individuare bug online. Se si modifica direttamente il codice e si salva, l'aggiornamento avviene immediatamente, il che potrebbe causare errori di sintassi e renderlo indisponibile. Il modo corretto sarebbe salvare il codice, quindi controllare la presenza di errori di sintassi con `php -l yourfile.php` e quindi eseguire il reload per aggiornare il codice.

Se gli sviluppatori desiderano effettivamente abilitare il monitoraggio dei file e l'aggiornamento automatico in modalità daemon, possono modificare il codice in Applications/FileMonitor/start.php, rimuovendo la parte di controllo di Worker::$daemonize.
