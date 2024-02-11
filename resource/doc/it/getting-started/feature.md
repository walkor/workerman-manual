# Caratteristiche di Workerman

### 1. Sviluppo in PHP puro
Le applicazioni sviluppate con Workerman possono essere eseguite in modo indipendente senza dipendere da php-fpm, apache o nginx. Ciò rende molto conveniente per gli sviluppatori PHP sviluppare, distribuire e debuggare le applicazioni.

### 2. Supporto per il multiprocessamento in PHP
Per sfruttare appieno le prestazioni dei server multi-CPU, Workerman supporta di default il multiprocessamento. Workerman avvia un processo principale e vari processi figlio per fornire servizio esterno, con il processo principale responsabile del monitoraggio dei processi figlio. Il modello di processo semplice rende Workerman più stabile ed efficiente.

### 3. Supporto per TCP e UDP
Workerman supporta due protocolli di trasporto, TCP e UDP. È sufficiente modificare un attributo per cambiare il protocollo di trasporto, senza dover modificare il codice di business.

### 4. Supporto per le connessioni persistenti
Spesso le applicazioni PHP devono mantenere connessioni persistenti con i client, ad esempio nelle chat o nei giochi. Tuttavia, i tradizionali contenitori PHP (apache, nginx, php-fpm) fanno fatica a gestire questo tipo di connessione. Utilizzando Workerman, è possibile utilizzare le connessioni persistenti fintanto che il servizio non chiude attivamente le connessioni. Un singolo processo Workerman può gestire decine di migliaia di connessioni simultanee, mentre i processi multipli possono gestire centinaia di migliaia o anche milioni di connessioni simultanee.

### 5. Supporto per vari protocolli di livello applicativo
Workerman supporta vari protocolli di livello applicativo, inclusi protocolli personalizzati. Cambiare il protocollo con Workerman è molto semplice: basta configurare un campo e il protocollo cambierà automaticamente, senza dover modificare il codice di business. È persino possibile aprire più porte con protocolli diversi per soddisfare le esigenze dei client diversi.

### 6. Supporto per l'alta concorrenza
Workerman supporta la libreria di polling degli eventi Libevent (richiede l'installazione dell'estensione event). Utilizzando Event per le connessioni persistenti ad alta concorrenza si ottiene un'eccellente performance. Se l'estensione Event non è installata, Workerman utilizza le chiamate di sistema Select integrate in PHP, ma consente comunque prestazioni estremamente potenti.

### 7. Supporto per il riavvio del servizio in modo fluido
Quando è necessario riavviare il servizio (ad esempio per il rilascio di una nuova versione), è fondamentale che i processi in esecuzione al momento del riavvio non vengano interrotti all'improvviso e che il riavvio non causi problemi di comunicazione ai client. Workerman offre la funzionalità di riavvio fluido, garantendo un aggiornamento del servizio senza intoppi.

### 8. Supporto per il monitoraggio e il caricamento automatico dei file
Nel processo di sviluppo, è importante che le modifiche al codice siano immediate, per poter visionare i risultati. Workerman offre il [componente di monitoraggio dei file](../components/file-monitor.md), in modo che i file vengano automaticamente ricaricati quando vengono aggiornati, consentendo così alle modifiche di essere immediatamente attive.

### 9. Supporto per l'esecuzione dei processi figlio con un utente specifico
Poiché i processi figlio sono quelli che gestiscono effettivamente le richieste degli utenti, è importante che abbiano un livello di autorizzazione adeguato per motivi di sicurezza. Workerman supporta l'esecuzione dei processi figlio con un utente specifico, garantendo così una maggiore sicurezza del server.

### 10. Supporto per la conservazione permanente degli oggetti o delle risorse
Workerman legge e interpreta i file PHP solo una volta durante l'esecuzione e li mantiene in memoria in modo permanente. Questo significa che le variabili statiche o le variabili globali all'interno del ciclo di vita di un processo vengono conservate in modo permanente, consentendo ad esempio di riutilizzare la stessa connessione al database per tutte le richieste gestite da un singolo processo, migliorando notevolmente l'efficienza dell'applicazione.

### 11. Alte prestazioni
Poiché i file PHP vengono letti e interpretati solo una volta e quindi memorizzati in memoria, si riduce notevolmente l'I/O del disco e la fase di inizializzazione, creazione e distruzione di richieste PHP. Inoltre, non essendo dipendente da nginx, apache o altri container, si riduce l'onere di comunicazione tra PHP e questi container. La conservazione permanente delle risorse riduce la necessità di reinizializzare connessioni al database e altro. Pertanto, lo sviluppo di applicazioni con Workerman garantisce prestazioni molto elevate.

### 12. Supporto per HHVM
Workerman è compatibile con HHVM (Hack Virtual Machine), garantendo un netto miglioramento delle prestazioni del PHP. In particolare, in attività di calcolo intensivo, le prestazioni sono eccellenti. Secondo test di stress effettuati, senza un carico di lavoro rilevante, Workerman in esecuzione su HHVM ha aumentato la velocità di trasferimento dei dati di rete del 30-80% rispetto all'esecuzione su Zend PHP 5.6.

### 13. Supporto per la distribuzione distribuita
### 14. Supporto per i processi in modalità demone
### 15. Supporto per l'ascolto su porte multiple
### 16. Supporto per il reindirizzamento dello standard input e output.
