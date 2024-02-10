# Visualizzazione dello stato di esecuzione

Esegui il comando ```php start.php status``` per visualizzare lo stato di esecuzione di WorkerMan, simile al seguente:

```plaintext
----------------------------------------------STATO GLOBALE---------------------------------------------------
Versione di Workerman: 3.5.13          Versione PHP: 5.5.9-1ubuntu4.24
Ora di avvio: 2018-02-03 11:48:20   in esecuzione da 112 giorni 2 ore   
Carico medio: 0, 0, 0            event-loop:\Workerman\Events\Event
4 workers       11 processi
Nome worker        exit_status      exit_count
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------STATO PROCESSO---------------------------------------------------
pid	memoria  ascolto                Nome worker        connessioni invio_fallito timer  richiesta_totale qps    stato
18306	2.25M   nessuno                ChatBusinessWorker 5           0         0       11            0      [inattivo]
18307	2.25M   nessuno                ChatBusinessWorker 5           0         0       8             0      [inattivo]
18308	2.25M   nessuno                ChatBusinessWorker 5           0         0       3             0      [inattivo]
18309	2.25M   nessuno                ChatBusinessWorker 5           0         0       14            0      [inattivo]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [inattivo]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [inattivo]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [inattivo]
18313	1.75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [inattivo]
18314	1.75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [inattivo]
18315	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [inattivo]
18316	1.5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [inattivo]
----------------------------------------------STATO PROCESSO---------------------------------------------------
Riepilogo	18M     -                        -                  54          0         4       138           0      [Riepilogo]
```

## Spiegazioni

### STATO GLOBALE

Dalla tabella sopra è possibile vedere:

Versione di Workerman ```version:3.5.13```

Ora di avvio ```2018-02-03 11:48:20```, in esecuzione da ```112 giorni 2 ore```

Carico medio del server ```load average: 0, 0, 0```, corrispondente al carico medio del sistema negli ultimi 1 minuto, 5 minuti e 15 minuti

Libreria di eventi utilizzata ```event-loop:\Workerman\Events\Event```

```4 workers``` (corrispondente a 3 tipi di processi, inclusi processi ChatGateway, ChatBusinessWorker, Register e WebServer)

```11 processi``` (totale di 11 processi)

```Nome worker``` (nome del processo worker)

```exit_status``` (codice di uscita del processo worker)

```exit_count``` (contatore di uscite con quel codice)

In generale, exit_status pari a 0 indica un'uscita corretta. Se diverso da zero, indica un'uscita anomala del processo, generando un messaggio di errore simile a `WORKER EXIT UNEXPECTED`, che verrà registrato nel file specificato in [Worker::logFile](worker/log-file.md).

**Ecco i significati più comuni di exit_status:**

* 0: indica un'uscita corretta, che si verifica spesso dopo un riavvio in modo soft durante un reload. È normale. Si noti che l'uso di exit o die nel codice può causare una terminazione con codice 0 e generare un messaggio di errore `WORKER EXIT UNEXPECTED`. Workerman non consente al codice di chiamare esplicitamente exit o die.
* 9: indica che il processo è stato ucciso dal segnale SIGKILL. Questo codice si verifica principalmente durante l'arresto o un riavvio in modo soft e si verifica quando un processo figlio non risponde entro il tempo previsto al segnale di riavvio inviato dal processo principale (ad esempio, a causa di un blocco prolungato, ad esempio, in attesa o in un loop di attesa attiva, come con MySQL, cURL, ecc.), momento in cui il processo principale lo termina forzatamente con il segnale SIGKILL. Si noti che l'uso del comando kill da riga di comando su Linux per inviare un segnale SIGKILL al processo figlio può generare lo stesso codice di uscita.
* 11: indica un core dump di PHP, generalmente causato dall'uso di estensioni instabili. Si consiglia di commentare le estensioni corrispondenti in php.ini. In alcuni casi potrebbe trattarsi di un bug in PHP, in tal caso è necessario aggiornare PHP.
* 65280: indica un errore fatale nel codice dell'applicazione, ad esempio chiamate di funzioni inesistenti o errori di sintassi. Le informazioni dettagliate sugli errori saranno registrate nel file specificato in [Worker::logFile](worker/log-file.md), oppure nel file specificato in [error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log) all'interno di php.ini (se specificato).
* 64000: indica un'eccezione generata dal codice dell'applicazione che non è stata gestita correttamente, causando la terminazione del processo. Se Workerman è in esecuzione in modalità debug, lo stack trace dell'eccezione sarà stampato sul terminale; altrimenti, verrà registrato nel file specificato in [Worker::stdoutFile](worker/stdout-file.md).

## STATO PROCESSO

pid: PID del processo

memoria: quantità attuale di memoria utilizzata dal processo (esclude la memoria utilizzata dal file eseguibile PHP stesso)

ascolto: protocollo di trasporto e indirizzo IP/porta in ascolto. Se non viene ascoltata alcuna porta, verrà visualizzato "nessuno". Vedere [Costruttore della classe Worker](worker/construct.md) per ulteriori dettagli.

Nome worker: nome del servizio eseguito dal processo, vedere [Attributo nome della classe Worker](worker/name.md)

connessioni: numero attuale di istanze di connessione TCP gestite dal processo. Questo valore è esatto e non cumulativo. Si noti che se un'istanza di connessione è chiusa, ma il contatore non diminuisce di conseguenza, potrebbe essere a causa dell'applicazione che mantiene l'oggetto $connection, impedendo così la distruzione dell'istanza di connessione.

richiesta_totale: numero totale di richieste ricevute dal processo dall'avvio fino a quel momento. Questo numero include non solo le richieste inviate dal client, ma anche le richieste di comunicazione interna a Workerman, come nel caso dell'architettura di GatewayWorker tra Gateway e BusinessWorker. Questo valore è cumulativo.

invio_fallito: numero di volte in cui l'invio di dati dal processo al client è fallito. Di solito, se questo valore è diverso da zero, indica un malfunzionamento del client, come la disconnessione. Vedi la pagina [status con invio_fallito](../faq/about-send-fail.md) per ulteriori informazioni. Questo valore è cumulativo.

timer: numero di timer attivi nel processo (esclusi i timer eliminati e quelli già scaduti). Si noti che questa funzionalità richiede Workerman versione >= 3.4.7. Questo valore è esatto e non cumulativo.

qps: numero di richieste di rete ricevute dal processo per secondo. Si noti che questo valore è incluso solo quando si usa ```-d``` con status e mostra 0 altrimenti. Questa funzionalità richiede Workerman versione >= 3.5.2. Questo valore è esatto e non cumulativo.

stato: stato attuale del processo, "inattivo" per indicare che è inattivo e "occupato" per indicare che è impegnato. Si noti che se un processo è brevemente occupato, questo è normale; tuttavia, se un processo rimane costantemente occupato, potrebbe essere bloccato da un'operazione di lunga durata o un'iterazione infinita nel codice dell'applicazione. In tal caso, consultare la sezione [Debugging Busy Process](busy-process.md) per risolvere il problema. Si noti che questa funzionalità richiede la versione di Workerman >= 3.5.0.

## Principio

Dopo l'esecuzione dello script status, il processo principale invierà un segnale ```SIGUSR2``` a tutti i processi worker. Successivamente, lo script status entrerà in una breve fase di sleep per attendere i risultati dello stato dei singoli processi worker. Durante questo periodo, i processi worker inattivi che ricevono il segnale ```SIGUSR2``` scriveranno immediatamente le proprie condizioni di esecuzione (ad esempio, numero di connessioni, numero di richieste, ecc.) su un file specifico sul disco, mentre i processi worker che stanno elaborando la logica aziendale aspetteranno di completare l'elaborazione prima di scrivere le proprie informazioni di stato. Dopo un breve periodo di sleep, lo script status inizierà a leggere le informazioni di stato dal disco e le visualizzerà nella console.

## Nota

Durante l'esecuzione di status, è possibile che alcuni processi mostrino uno stato "occupato" a causa dell'elaborazione di attività aziendali (ad esempio, lunghe attese su richieste di rete o database o esecuzione di grandi cicli), impedendo loro di inviare lo stato, causando così un'apparenza di occupazione. In tal caso, è necessario esaminare il codice aziendale per individuare e valutare l'eventuale blocco prolungato. Se non corrisponde alle aspettative, è necessario eseguire le operazioni di debug di cui alla sezione [Debugging Busy Process](busy-process.md).
