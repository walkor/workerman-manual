# Vérifier l'état d'exécution

Exécutez ```php start.php status```
pour vérifier l'état d'exécution de Workerman, similaire à ce qui suit :

```html
----------------------------------------------ETAT GLOBAL----------------------------------------------------
Version de Workerman : 3.5.13          Version de PHP : 5.5.9-1ubuntu4.24
heure de démarrage : 2018-02-03 11:48:20   en cours d'exécution depuis 112 jours 2 heures   
charge moyenne : 0, 0, 0            boucle d'événements : \Workerman\Events\Event
4 travailleurs       11 processus
nom_du_travailleur        statut_de_sortie      nombre_de_sorties
ChatBusinessWorker       0                0
ChatGateway              0                0
Register                 0                0
WebServer                0                0
----------------------------------------------STATUT DU PROCESSUS---------------------------------------------------
pid  mémoire  écoute                nom_du_travailleur        connexions envoi_échoué minuteurs  demande_totale qps    état
18306  2,25M   aucun                 ChatBusinessWorker 5           0         0       11            0      [inactif]
18307  2,25M   aucun                 ChatBusinessWorker 5           0         0       8             0      [inactif]
18308  2,25M   aucun                 ChatBusinessWorker 5           0         0       3             0      [inactif]
18309  2,25M   aucun                 ChatBusinessWorker 5           0         0       14            0      [inactif]
18310  2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [inactif]
18311  2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [inactif]
18312  2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [inactif]
18313  1,75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [inactif]
18314  1,75M   texte://0.0.0.0:1236      Register           8           0         0       8             0      [inactif]
18315  1,5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [inactif]
18316  1,5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [inactif]
----------------------------------------------STATUT DU PROCESSUS---------------------------------------------------
Résumé  18M     -                        -                  54          0         4       138           0      [Résumé]
```

## Explication

### ETAT GLOBAL

Dans cette colonne, nous pouvons voir :

Version de Workerman ```version:3.5.13```

Heure de démarrage```2018-02-03 11:48:20```, en cours d'exécution depuis ```112 jours 2 heures ```

Charge du serveur ```charge moyenne: 0, 0, 0```, représentant la charge moyenne du système au cours des 1, 5 et 15 dernières minutes.

Utilisation de la bibliothèque d'événements IO, ```boucle d'événements : \Workerman\Events\Event```

 ```4 travailleurs``` (3 types de processus, y compris ChatGateway, ChatBusinessWorker, processus Register, processus WebServer)

 ``` 11 processus``` (au total 11 processus)

 ``` nom_du_travailleur``` (nom du processus de travail)

 ``` statut_de_sortie``` (code de sortie du processus de travail)

 ``` nombre_de_sorties ``` (nombre de sorties avec ce code)

En général, un statut_de_sortie de 0 indique une sortie normale. Toute autre valeur indique une sortie anormale du processus, générant un message d'erreur similaire à ```SORTIE DU TRAVAILLEUR INATTENDUE```. Ces messages d'erreur seront enregistrés dans le fichier spécifié dans [Worker::logFile](worker/log-file.md).

**Les statut_de_sortie courants et leurs significations sont les suivants :**

* 0 : sortie normale, apparaîtra après un redémarrage lisse. Notez que l'appel à exit ou die dans le code entraînera également une sortie avec le code 0 et génèrera un message d'erreur ```SORTIE DU TRAVAILLEUR INATTENDUE```. Dans Workerman, il est interdit d'utiliser les instructions exit ou die dans le code métier.
* 9 : le processus a été tué par un signal SIGKILL. Ce code de sortie se produit principalement lors de l'arrêt ou du redémarrage lisse, suite à un manque de réponse du sous-processus au signal de redémarrage du processus principal (par exemple, en raison de blocages prolongés tels que mysql ou curl, ou boucles infinies dans le code métier), qui est ensuite forcé de se terminer par un signal SIGKILL. Notez que l'utilisation de la commande kill dans la ligne de commande Linux pour envoyer un signal SIGKILL au sous-processus entraînera également ce code de sortie.
* 11 : indique qu'il y a eu un core dump de PHP, généralement dû à l'utilisation de extensions instables. Dans ce cas, il est recommandé de commenter la ligne correspondante dans le fichier php.ini. Dans de rares cas, il peut s'agir d'un bogue de PHP, nécessitant alors une mise à jour de PHP.
* 65280 : le code de sortie est dû à une erreur fatale dans le code métier, par exemple, l'appel d'une fonction inexistante, une erreur de syntaxe, etc. Les informations spécifiques sur l'erreur seront enregistrées dans le fichier spécifié dans [Worker::logFile](worker/log-file.md) ou dans le fichier [error_log](https://php.net/manual/zh/errorfunc.configuration.php#ini.error-log) du fichier [php.ini](https://php.net/manual/zh/ini.list.php) (s'il est spécifié).
* 64000 : le code de sortie est dû à une exception non gérée dans le code métier, causant la sortie du processus. Si Workerman fonctionne en mode de débogage, la pile d'appels d'exception sera affichée dans le terminal, sinon elle sera enregistrée dans le fichier spécifié dans [Worker::stdoutFile](worker/stdout-file.md).

## STATUT DU PROCESSUS

pid : ID du processus

mémoire : quantité de mémoire actuellement utilisée par le processus (hors mémoire utilisée par le fichier exécutable PHP)

écoute : protocole de transport et adresse IP sur laquelle le processus écoute. Si aucune écoute n'est en cours, la mention "aucun" s'affiche. Voir [Constructeur de classe Worker](worker/construct.md) pour plus de détails.

nom_du_travailleur : nom du service en cours d'exécution par le processus. Voir la propriété name de la classe Worker (worker/name.md).

connexions : nombre **actuel** d'instances de connexion TCP pour ce processus. Les instances de connexion comprennent les instances TcpConnection et AsyncTcpConnection. Cette valeur est en temps réel et n'est pas cumulative. Remarque : si une instance de connexion est fermée à l'aide de la méthode close, mais que le compteur correspondant n'est pas décrémenté, cela peut signifier que le code métier a conservé l'objet $connection, empêchant ainsi la destruction de l'instance de connexion.

demande_totale : nombre total de requêtes reçues par ce processus depuis son démarrage. Ce nombre inclut les demandes des clients ainsi que les requêtes internes de Workerman, par exemple les requêtes de communication entre Gateway et BusinessWorker dans l'architecture GatewayWorker. Ce nombre est cumulatif.

envoi_échoué : nombre de tentatives d'envoi de données aux clients ayant échoué. Les échecs surviennent généralement lorsque la connexion du client est interrompue. Un nombre différent de zéro est généralement considéré comme normal, voir [Raisons des échecs d'envoi dans le statut](../faq/about-send-fail.md). Ce nombre est cumulatif.

minuteurs : nombre d'instances de minuterie actives pour ce processus (à l'exclusion des minuteries supprimées ou des minuteries à exécution unique déjà expirées). Notez que cette fonctionnalité nécessite une version de Workerman >=3.4.7. Cette valeur est en temps réel et n'est pas cumulative.

qps: nombre de requêtes réseaux reçues par seconde pour ce processus. Remarque : seules les commandes ```php start.php status -d``` incluront cette statistique, sinon elle affichera 0. Cette fonctionnalité nécessite une version de Workerman >=3.5.2. Cette valeur est en temps réel et n'est pas cumulative.

état : état du processus, "inactif" signifie que le processus est inactif, "occupé" signifie qu'il est occupé. Remarque : si un processus est brièvement occupé, c'est normal, mais s'il reste constamment occupé, cela peut être dû à un blocage du code métier ou à une boucle infinie. Consultez la section [Débogage d'un processus occupé](busy-process.md) pour plus de détails. Cette fonctionnalité nécessite une version de Workerman >=3.5.0.

## Principes
Lors de l'exécution du script status, le processus principal envoie un signal ```SIGUSR2``` à tous les processus de travail. Ensuite, le script status entre dans une brève période de sommeil pour attendre que les résultats de l'état de tous les processus de travail soient rassemblés. Pendant ce temps, les processus de travail inactifs, recevant le signal ```SIGUSR2```, écrivent immédiatement leurs états actuels (nombre de connexions, nombre de demandes, etc.) dans des fichiers spécifiques sur le disque, tandis que les processus de travail occupés attendent la fin du traitement de leur logique métier avant d'écrire leurs états. Après cette courte période de sommeil, le script status commence à lire les fichiers d'état sur le disque et affiche les résultats dans la console.

## Remarques
Lors de l'exécution de la commande status, il est possible de constater que certains processus sont indiqués comme étant occupés. Cela est dû au fait que le processus est occupé à traiter des tâches métier (telles qu'un blocage prolongé dans une requête curl ou une base de données, ou l'exécution d'une grande boucle), rendant impossible la transmission de l'état et affichant le processus comme étant occupé.

Pour résoudre ce problème, il convient de vérifier le code métier pour déterminer la source du blocage et d'évaluer si le temps d'attente est conforme aux attentes. Si ce n'est pas le cas, des mesures doivent être prises pour déboguer le code métier, comme indiqué dans la section [Débogage d'un processus bloqué](busy-process.md).
