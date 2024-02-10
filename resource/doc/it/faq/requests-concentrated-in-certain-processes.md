# Richiesta concentrata in alcuni processi

### Fenomeno
A volte quando controlliamo con il comando `php start.php status`, notiamo che le richieste sono concentrate in alcuni processi specifici, mentre gli altri processi rimangono completamente inattivi.

### Meccanismo di prelazione
Workerman gestisce le connessioni in modo **prelazionale** di default, il che significa che quando un client avvia una connessione, tutti i processi inattivi hanno l'opportunità di ottenere quella connessione, e il più veloce vince. La decisione su chi è più veloce è presa dallo scheduler del kernel del sistema operativo. Il sistema operativo potrebbe privilegiare il processo che è stato utilizzato di recente per l'uso della CPU, poiché potrebbe ancora avere le informazioni sul contesto del processo precedente nei registri della CPU, riducendo così i costi del cambio del contesto. Quindi, quando il business è sufficientemente veloce o durante il test delle prestazioni, è più probabile che si verifichi la concentrazione delle connessioni in alcuni processi, poiché questa strategia evita cambi frequenti di processo, e di solito offre le migliori prestazioni.

### Meccanismo di rotazione
Workerman può modificare il modo in cui le connessioni vengono acquisite impostando `$worker->reusePort = true;` in modo da utilizzare il meccanismo di acquisizione delle connessioni in modo **rotativo**, in cui il kernel distribuisce le connessioni in modo approssimativamente uniforme tra tutti i processi, in modo che tutti i processi lavorino insieme per gestire le richieste.

### Malintesi
Molti sviluppatori credono che migliori le prestazioni se tutti i processi partecipano alla gestione delle richieste, ma in realtà non è sempre così. Quando il business è sufficientemente semplice, maggiore è il numero di processi che partecipano alla gestione delle richieste, maggiore è il throughput del server. Ad esempio, su un server a 4 core, spesso il QPS è massimo quando il numero di processi è impostato su 4 in un semplice test di prestazioni. Se il numero di processi coinvolti nella gestione supera di molto il numero di core della CPU, l'overhead del contesto del processo aumenta e di conseguenza le prestazioni peggiorano. In generale, quando è coinvolto un business con database, è possibile che le prestazioni siano migliori impostando il numero di processi tra 3 e 6 volte il numero di core della CPU.
