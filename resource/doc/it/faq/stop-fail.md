# Errore di arresto

## Sintomo:
Eseguendo ```php start.php stop``` appare il messaggio ```stop fail```.

### Prima possibilità
Se si avvia Workerman in modalità debug e si preme ```ctrl z``` nella console, viene inviato un segnale ```SIGSTOP``` a Workerman, facendolo entrare in modalità di sospensione in background, impedendo così di rispondere al comando di stop (```SIGINT```).
**Soluzione:**
Nella console in cui è in esecuzione Workerman, digitare ```fg``` (per inviare il segnale ```SIGCONT```) e premere Invio, per riportare Workerman in primo piano, quindi premere ```ctrl c``` (per inviare il segnale ```SIGINT```) e arrestare Workerman.
Se non si riesce a fermarlo, provare a eseguire i seguenti comandi:
```
killall -9 php
```
```
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

### Seconda possibilità
L'utente che tenta di arrestare Workerman non corrisponde all'utente che lo ha avviato, cioè l'utente di arresto non ha i permessi per fermare Workerman.
**Soluzione:**
Passare all'utente che ha avviato Workerman, o utilizzare un utente con privilegi più elevati per arrestare Workerman.

### Terza possibilità
Il file pid del processo principale di Workerman è stato eliminato, quindi lo script non può trovare il processo pid, causando un fallimento nell'arresto.
**Soluzione:**
Salvare il file pid in una posizione sicura, consultare il manuale [Worker::$pidFile](../worker/pid-file.md).

### Quarta possibilità
Il file pid del processo principale di Workerman non corrisponde al processo di Workerman.
**Soluzione:**
Aprire il file pid del processo principale di Workerman, verificare il pid del processo principale, che di default si trova nella stessa directory di Workerman. Eseguire il comando ```ps aux | grep pid del processo principale``` per verificare se il processo corrisponde a Workerman. Se non corrisponde, potrebbe essere dovuto a un riavvio del server che ha reso obsoleto il pid salvato da Workerman e quel pid è stato utilizzato da un altro processo, causando il fallimento dell'arresto. In questo caso, eliminare il file pid.

### Quinta possibilità
Se si è installata l'estensione grpc ma non sono stati impostati i relativi parametri dell'ambiente, all'avvio verrà creato un processo di montaggio aggiuntivo, causando un fallimento nell'arresto.

