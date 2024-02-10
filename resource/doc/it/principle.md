# Principio

### Descrizione del Worker
Il Worker è il contenitore più fondamentale in WorkerMan, può aprire più processi per ascoltare le porte e comunicare utilizzando protocolli specifici, simile a nginx che ascolta su una certa porta. Ogni processo Worker funziona in modo indipendente, utilizzando Epoll (richiede l'installazione dell'estensione event) e IO non bloccante. Ogni processo Worker può gestire decine di migliaia di connessioni client e gestire i dati inviati da queste connessioni. Il processo principale si occupa di monitorare i processi figlio per garantire la stabilità, senza ricevere dati o eseguire alcuna logica di business.

### Relazione tra il cliente e il processo Worker
![workerman master woker](images/Worker.png)

### Relazione tra il processo principale e il processo Worker
![workerman master woker](images/Worker2.png)

**Caratteristiche:**

Dalla figura possiamo vedere che ogni Worker mantiene le proprie connessioni client, consentendo facilmente di implementare la comunicazione in tempo reale tra client e server. Basandosi su questo modello, possiamo facilmente soddisfare alcune esigenze di sviluppo di base, come server HTTP, server Rpc, invio in tempo reale dei dati da dispositivi intelligenti, invio di dati lato server, server di gioco, backend di applicazioni WeChat e così via.
