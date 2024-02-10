# Modifiche non in vigore dopo il codice

**Reason:**

Workerman è in esecuzione in memoria permanente, il che evita di leggere ripetutamente dal disco e di interpretare e compilare il codice PHP ripetutamente al fine di ottenere le massime prestazioni. Pertanto, le modifiche al codice di business devono essere ricaricate manualmente o riavviate per entrare in vigore.

Allo stesso tempo, Workerman fornisce un servizio di monitoraggio dell'aggiornamento dei file, che rileva automaticamente le modifiche ai file e avvia automaticamente la ricarica, ricaricando i file PHP. I sviluppatori dovrebbero includerlo nel progetto e farlo avviare con il progetto.

Nota: il sistema Windows non supporta la ricarica e non può utilizzare il servizio di monitoraggio.

**Indirizzi di download del servizio di monitoraggio dei file:**

1. Versione indipendente: https://github.com/walkor/workerman-filemonitor

2. Versione con dipendenza inotify: https://github.com/walkor/workerman-filemonitor-inotify

**Differenza tra le due versioni:**

La versione 1 utilizza il metodo del polling dell'ora di aggiornamento del file per verificare se il file è stato aggiornato, mentre la versione 2 sfrutta il meccanismo del kernel Linux [inotify](https://baike.baidu.com/view/2645027.htm), con il quale il sistema notifica attivamente a Workerman quando un file viene aggiornato.

Di solito, la versione 1 senza dipendenze è sufficiente.
