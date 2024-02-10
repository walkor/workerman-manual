# Alcune domande importanti per gli sviluppatori di Workerman

**1. Limitazioni dell'ambiente Windows**

In un sistema Windows, un singolo processo in Workerman supporta solo 200+ connessioni. Non è possibile utilizzare il parametro di conteggio per impostare processi multipli. I comandi come status, stop, reload, restart non possono essere utilizzati in un sistema Windows. Non è possibile eseguire il processo come servizio in background; il servizio si interrompe quando la finestra di comando viene chiusa. Non è possibile inizializzare più di una connessione in un singolo file. Per evitare queste limitazioni, si consiglia di utilizzare un sistema Linux nell'ambiente di produzione e Windows nell'ambiente di sviluppo.

**2. Workerman non dipende da apache o nginx**

Workerman è già un contenitore simile ad apache/nginx e può essere eseguito purché l'ambiente PHP sia funzionante.

**3. Workerman viene avviato da riga di comando**

L'avvio è simile a quello di Apache mediante un comando da riga di comando (di solito non è possibile utilizzarlo nello spazio web). L'interfaccia di avvio è simile a quella mostrata di seguito:
![](image/screenshot_1495622774534.png)

**4. È necessario aggiungere un battito cardiaco per le connessioni a lungo termine**

È estremamente importante aggiungere un battito cardiaco alle connessioni a lungo termine, in quanto la mancanza di comunicazione a lungo termine può portare al rilascio della connessione da parte del nodo di routing, causandone la chiusura. Per ulteriori informazioni sul battito cardiaco in Workerman consultare [Spiegazione del battito cardiaco di Workerman](faq/heartbeat.md) e [Spiegazione del battito cardiaco di gatewayWorker](https://www.workerman.net/doc/gateway-worker/heartbeat.html).

**5. La corrispondenza dei protocolli tra client e server è essenziale per la comunicazione**

Questo è un problema comune per gli sviluppatori. Ad esempio, se il client utilizza il protocollo websocket, anche il server deve utilizzare lo stesso protocollo (ad esempio, il server ```new Worker('websocket://0.0.0.0...')```), altrimenti non sarà possibile stabilire una connessione e comunicare. Non cercate di accedere alla porta del protocollo websocket tramite la barra degli indirizzi del browser e non cercate di utilizzare il protocollo websocket per accedere alla porta del protocollo TCP raw; è fondamentale che i protocolli siano corrispondenti. 

È simile al concetto di dover parlare inglese se si vuole comunicare con una persona inglese, mentre se si vuole comunicare con una persona giapponese, bisogna usare il giapponese. In questo caso, il linguaggio è simile ai protocolli di comunicazione e entrambe le parti (client e server) devono utilizzare lo stesso "linguaggio" per comunicare.

**6. Possibili cause di fallimento della connessione**

Un problema comune all'inizio dell'utilizzo di Workerman è il fallimento della connessione del client al server. Le cause più comuni sono:

1. Il firewall del server (inclusi i gruppi di sicurezza dei server cloud) blocca la connessione (50% di probabilità).
2. Il client e il server utilizzano protocolli non corrispondenti (30% di probabilità).
3. Sbagliato IP o porta (15% di probabilità).
4. Il server non è stato avviato.

**7. Evitare di utilizzare le istruzioni exit, die e sleep**

L'uso di istruzioni exit e die nel codice aziendale causerà l'uscita del processo e mostrerà un errore "WORKER EXIT UNEXPECTED". Naturalmente, quando un processo termina, ne verrà immediatamente avviato uno nuovo per continuare a fornire il servizio. Se è necessario il ritorno, è possibile utilizzare l'istruzione return. L'istruzione sleep farà sì che il processo entri in modalità sleep, durante la quale non verrà eseguito alcun servizio, causando l'interruzione del framework e l'incapacità di gestire le richieste dei client di quel processo.

**8. Evitare di utilizzare la funzione pcntl_fork**

La funzione `pcntl_fork` viene utilizzata per creare dinamicamente nuovi processi. Se viene utilizzata nel codice aziendale, potrebbe causare la generazione di processi orfani non recuperabili, portando a eccezioni nel servizio. L'uso di `pcntl_fork` nel codice aziendale influenzerà anche l'elaborazione di eventi come le connessioni, i messaggi, la chiusura delle connessioni e i timer, causando eccezioni impreviste.

**9. Evitare i cicli infiniti nel codice aziendale**

Nel codice aziendale, evitate i cicli infiniti in quanto ciò potrebbe impedire al framework di Workerman di gestire le richieste di altri client, causando un blocco del controllo.

**10. Riavviare Workerman dopo la modifica del codice**

Workerman è un framework residente in memoria e deve essere riavviato per vedere gli effetti del nuovo codice.

**11. Utilizzo consigliato di GatewayWorker per le applicazioni a lungo termine**

Molti sviluppatori utilizzano Workerman per sviluppare applicazioni a **lungo termine** come comunicazione istantanea, Internet delle cose, ecc. Si consiglia di utilizzare direttamente il framework GatewayWorker, sviluppato appositamente per semplificare e migliorare l'implementazione delle applicazioni a lungo termine su Workerman.

**12. Supporto per una maggiore concorrenza**

Se il numero di connessioni concorrenti supera 1000, è vivamente consigliato [ottimizzare il kernel di Linux](appendices/kernel-optimization.md) e [installare l'estensione event](appendices/install-extension.md).
