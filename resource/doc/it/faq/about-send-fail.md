# Motivo del fallimento dell'invio nello stato

**Sintomo:**

Eseguendo il comando `status`, si nota la presenza di situazioni di `send_fail`. Qual è la ragione di ciò?

**Risposta:**

In genere, il `send_fail` non è un grosso problema e di solito è causato dalla chiusura attiva della connessione da parte del client o dall'incapacità del client di ricevere i dati, che porta al fallimento dell'invio dei dati.

Ci sono due ragioni per il `send_fail`:

1. Quando si usa l'interfaccia `send` per inviare dati al client e si scopre che il client è disconnesso, il contatore di `send_fail` aumenta di 1. Poiché si tratta di una disconnessione attiva del client, è considerato un comportamento normale e di solito può essere ignorato.

2. La velocità di invio dei dati dal server è superiore alla velocità di ricezione del client, il che porta all'accumulo continuo dei dati nel buffer di invio del server (workerman stabilisce un buffer di invio per ciascun client). Se la dimensione del buffer supera il limite (di default, `TcpConnection::$maxSendBufferSize` è 1 MB), i dati saranno scartati, attivando l'evento `onError` (se presente) e aumentando il contatore di `send_fail`.

Ad esempio, se si riduce al minimo il browser, JavaScript potrebbe sospendere l'esecuzione, causando la pausa nella ricezione dei dati dal server. Se i dati si accumulano nel buffer per un lungo periodo, superando il limite, ogni chiamata a `send` aumenterà il contatore di `send_fail`.

**Conclusione:**

Di solito non c'è bisogno di preoccuparsi del `send_fail` causato dalla disconnessione del client.

Se il `send_fail` è causato dalla lentezza del client nel ricevere i dati, è necessario verificare lo stato del client.

Se la velocità di ricezione del client è **costantemente** inferiore rispetto alla velocità di invio del server, è necessario valutare l'ottimizzazione del flusso di lavoro o delle prestazioni del client. Se il problema è legato alla larghezza di banda, si può considerare l'aumento della larghezza di banda del server.
