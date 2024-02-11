# Suivi des appels système

Lorsque vous souhaitez savoir ce que fait un processus, vous pouvez suivre tous ses appels système avec la commande ```strace```.

1. Exécuter ```php start.php status``` permet de voir les informations des processus Workerman comme ci-dessous :

```plaintext
Bonjour admin
---------------------------------------GLOBAL STATUS--------------------------------------------
Version de Workerman : 3.0.1
heure de démarrage : 2014-08-12 17:42:04   en cours d'exécution depuis 0 jours 1 heure
charge moyenne : 3.34, 3.59, 3.67
1 utilisateur          8 ouvriers       14 processus
nom du ouvrier       état de sortie     compte de sorties
BusinessWorker    0                0
ChatWeb           0                0
FileMonitor       0                0
Gateway           0                0
Monitor           0                0
StatisticProvider 0                0
StatisticWeb      0                0
StatisticWorker   0                0
---------------------------------------PROCESS STATUS-------------------------------------------
pid	mémoire      en écoute        horodatage  nom du ouvrier       demande totale erreur de paquet orage de tonnerre fermeture client échec d'envoi jeter une exception réussi / total
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
```

2. Par exemple, si vous voulez savoir ce que fait le processus Gateway avec l'identifiant de processus (PID) 10354, vous pouvez exécuter la commande suivante : ```strace -p 10354``` (peut nécessiter des privilèges root), similaire à ce qui suit :

```plaintext
sudo strace -p 10354
Processus 10354 attaché - appuyez sur interrupt pour quitter
clock_gettime(CLOCK_MONOTONIC, {118627, 242986712}) = 0
gettimeofday({1407840609, 102439}, NULL) = 0
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Appel système interrompu)
--- SIGUSR2 (Signal utilisateur défini 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (masque maintenant [])
clock_gettime(CLOCK_MONOTONIC, {118627, 699623319}) = 0
gettimeofday({1407840609, 559092}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118627, 699810499}) = 0
gettimeofday({1407840609, 559277}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Ressource temporairement non disponible)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Appel système interrompu)
--- SIGUSR2 (Signal utilisateur défini 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (masque maintenant [])
clock_gettime(CLOCK_MONOTONIC, {118628, 699497204}) = 0
gettimeofday({1407840610, 558937}, NULL) = 0
epoll_wait(3, {{EPOLLIN, {u32=9, u64=9}}}, 32, -1) = 1
clock_gettime(CLOCK_MONOTONIC, {118628, 699588603}) = 0
gettimeofday({1407840610, 559023}, NULL) = 0
recv(9, "\f", 1024, 0)                  = 1
recv(9, 0xb60b4880, 1024, 0)            = -1 EAGAIN (Ressource temporairement non disponible)
epoll_wait(3, 985f4f0, 32, -1)          = -1 EINTR (Appel système interrompu)
--- SIGUSR2 (Signal utilisateur défini 2) @ 0 (0) ---
send(7, "\f", 1, 0)                     = 1
sigreturn()                             = ? (masque maintenant [])
```

3. Chaque ligne représente un appel système, à partir de ces informations, il est facile de voir ce que fait le processus, de localiser où il est bloqué, s'il est bloqué sur une connexion ou lors de la lecture de données réseau, etc.
