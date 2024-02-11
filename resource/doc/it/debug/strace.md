# Tracciare le chiamate di sistema

Quando si desidera sapere cosa sta facendo un processo, è possibile tracciare tutte le chiamate di sistema di un processo utilizzando il comando ```strace```.

1. Eseguendo il comando php start.php status è possibile visualizzare le informazioni sui processi correlati a Workerman come segue:

```
Ciao amministratore
---------------------------------------STATO GLOBALE--------------------------------------------
Versione Workerman: 3.0.1
orario di avvio: 2014-08-12 17:42:04   attivo da 0 giorni 1 ora
carico medio: 3,34, 3,59, 3,67
1 utenti         8 worker       14 processi
nome_worker       stato_uscita     conteggio_uscite
BusinessWorker    0                0
ChatWeb           0                0
FileMonitor       0                0
Gateway           0                0
Monitor           0                0
StatisticProvider 0                0
StatisticWeb      0                0
StatisticWorker   0                0
---------------------------------------STATO PROCESSO-------------------------------------------
pid	memoria      in_ascolto        timestamp  nome_worker       richiesta_totale errore_pacchetto thunder_herd chiusura_client invio_fallito lancio_eccezione con successo/totale
10352	1.5M    tcp://0.0.0.0:55151  1407836524 ChatWeb           12             0          0            2            0         0               100%
10354	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          0            0            0         0               100%
10355	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           0              0          1            0            0         0               100%
10365	1.25M   tcp://0.0.0.0:55757  1407836524 StatisticWeb      0              0          0            0            0         0               100%
10358	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10364	1.25M   tcp://0.0.0.0:55858  1407836524 StatisticProvider 0              0          0            0            0         0               100%
10356	1.25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10366	1.25M   udp://0.0.0.0:55656  1407836524 StatisticWorker   55             0          0            0            0         0               100%
10349	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10350	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    0              0          0            0            0         0               100%
10351	1.5M    tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10348	1.25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    2              0          0            0            0         0               100%
```

2. Ad esempio, se desideriamo sapere cosa sta facendo il processo gateway con pid 10354, possiamo eseguire il comando strace -p 10354 (potrebbe essere necessario il permesso di root) come segue:

```
sudo strace -p 10354
Processo 10354 collegato - interrompere per uscire
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Chiamata di sistema interrotta)
--- SIGUSR2 (Segnale definito dall'utente 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (maschera ora [])
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Risorsa temporaneamente non disponibile)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Chiamata di sistema interrotta)
--- SIGUSR2 (Segnale definito dall'utente 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (maschera ora [])
clock_gettime(CLOCK_MONOTONIC, {118628, 699497204}) = 0
gettimeofday({1407840610, 558937}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118628, 699588603}) = 0
gettimeofday({1407840610, 559023}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Risorsa temporaneamente non disponibile)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Chiamata di sistema interrotta)
--- SIGUSR2 (Segnale definito dall'utente 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (maschera ora [])
```

3. Ogni riga rappresenta una chiamata di sistema, da queste informazioni è facile vedere cosa sta facendo il processo e individuare dove si sta bloccando, se è bloccato su una connessione o nella lettura dei dati di rete, ecc.
