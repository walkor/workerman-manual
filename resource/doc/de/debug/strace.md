# Verfolgung von Systemaufrufen

Wenn Sie wissen möchten, was ein Prozess gerade tut, können Sie mit dem Befehl ```strace``` alle Systemaufrufe eines Prozesses verfolgen.

1. Durch die Ausführung von ```php start.php status``` können Sie Informationen zu den Workerman-bezogenen Prozessen wie folgt anzeigen:

````
Hallo Administrator
---------------------------------------GLOBALER STATUS--------------------------------------------
WorkerMan-Version: 3.0.1
Startzeit: 2014-08-12 17:42:04   Laufzeit 0 Tage 1 Stunde
Durchschnittliche Auslastung: 3,34, 3,59, 3,67
1 Benutzer          8 Worker       14 Prozesse
Worker-Name       Exit-Status     Exit-Count
BusinessWorker    0                0
ChatWeb           0                0
FileMonitor       0                0
Gateway           0                0
Monitor           0                0
StatisticProvider 0                0
StatisticWeb      0                0
StatisticWorker   0                0
---------------------------------------PROZESS-STATUS-------------------------------------------
Pid	Speicher      Listening        Timestamp  Worker-Name       Gesamt-Anfragen Paketfehler Blitzherd Client-Schließung Sendefehler Ausnahme-Auslösung Erfolg/Insgesamt
10352	1,5M    tcp://0.0.0.0:55151  1407836524 ChatWeb           12             0          0            2            0         0               100%
10354	1,25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          0            0            0         0               100%
10355	1,25M   tcp://0.0.0.0:7272   1407836524 Gateway           0              0          1            0            0         0               100%
10365	1,25M   tcp://0.0.0.0:55757  1407836524 StatisticWeb      0              0          0            0            0         0               100%
10358	1,25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10364	1,25M   tcp://0.0.0.0:55858  1407836524 StatisticProvider 0              0          0            0            0         0               100%
10356	1,25M   tcp://0.0.0.0:7272   1407836524 Gateway           3              0          2            0            0         0               100%
10366	1,25M   udp://0.0.0.0:55656  1407836524 StatisticWorker   55             0          0            0            0         0               100%
10349	1,25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10350	1,25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    0              0          0            0            0         0               100%
10351	1,5M    tcp://127.0.0.1:7373 1407836524 BusinessWorker    5              0          0            0            0         0               100%
10348	1,25M   tcp://127.0.0.1:7373 1407836524 BusinessWorker    2              0          0            0            0         0               100%
````

2. Wenn Sie beispielsweise wissen möchten, was der Gateway-Prozess mit der PID 10354 gerade macht, können Sie den Befehl ```strace -p 10354``` ausführen (möglicherweise benötigen Sie Root-Rechte):

````
sudo strace -p 10354
Prozess 10354 angehängt - Unterbrechung zum Beenden
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Unterbrochener Systemaufruf)
--- SIGUSR2 (Signal 2 vom Benutzer definiert) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (jetzt Maske [])
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Ressource vorübergehend nicht verfügbar)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Unterbrochener Systemaufruf)
--- SIGUSR2 (Signal 2 vom Benutzer definiert) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (jetzt Maske [])
clock_gettime(CLOCK_MONOTONIC, {118628, 699497204}) = 0
gettimeofday({1407840610, 558937}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118628, 699588603}) = 0
gettimeofday({1407840610, 559023}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Ressource vorübergehend nicht verfügbar)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Unterbrochener Systemaufruf)
--- SIGUSR2 (Signal 2 vom Benutzer definiert) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (jetzt Maske [])
````

3. Jede Zeile entspricht einem Systemaufruf. Aus diesen Informationen können wir leicht erkennen, was der Prozess gerade macht. Wir können herausfinden, wo der Prozess hängt, ob es sich um eine Verbindung handelt oder ob er Netzwerkdaten liest.
