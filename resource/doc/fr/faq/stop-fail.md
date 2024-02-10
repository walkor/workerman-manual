# Arrêt échoué

## Phénomène :
L'exécution de ```php start.php stop``` affiche ```stop fail```.

### Première possibilité
Si Workerman a été lancé en mode débogage, et que le développeur a envoyé un signal ```SIGSTOP``` à Workerman en appuyant sur ```ctrl z``` dans le terminal, cela aura entraîné la mise en arrière-plan et la suspension de Workerman, l'empêchant ainsi de répondre à la commande d'arrêt (signal ```SIGINT```).

**Solution :**
Dans le terminal où Workerman a été lancé, tapez ```fg``` (pour envoyer le signal ```SIGCONT```) puis appuyez sur Entrée pour ramener Workerman à l'avant-plan, puis appuyez sur ```ctrl c``` (pour envoyer le signal ```SIGINT```) pour arrêter Workerman.

Si cela ne fonctionne pas, essayez d'exécuter les deux commandes suivantes :
```
killall -9 php
```
```
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

### Deuxième possibilité
L'utilisateur exécutant la commande d'arrêt n'est pas le même que celui ayant lancé Workerman, c'est-à-dire que l'utilisateur d'arrêt n'a pas les autorisations nécessaires pour arrêter Workerman.

**Solution :**
Changez d'utilisateur pour celui ayant lancé Workerman, ou utilisez un utilisateur ayant des autorisations plus élevées pour arrêter Workerman.

### Troisième possibilité
Le fichier pid du processus principal de Workerman a été supprimé, ce qui fait que le script ne trouve pas le processus pid, provoquant ainsi un échec de l'arrêt.

**Solution :**
Sauvegardez le fichier pid dans un emplacement sécurisé, consultez le manuel [Worker::$pidFile](../worker/pid-file.md).

### Quatrième possibilité
Le processus correspondant au fichier pid du processus principal de Workerman n'est pas celui de Workerman.

**Solution :**
Ouvrez le fichier pid du processus principal de Workerman pour consulter son pid, le fichier pid se trouve généralement dans le répertoire parallèle à celui de Workerman. Exécutez la commande ```ps aux | grep pid du processus principal``` pour vérifier si le processus correspondant est bien celui de Workerman. Si ce n'est pas le cas, il est possible que le serveur ait redémarré, ce qui a rendu obsolète le pid enregistré par Workerman, et ce pid est maintenant utilisé par un autre processus, provoquant ainsi un échec de l'arrêt. Si tel est le cas, supprimez simplement le fichier pid.

### Cinquième possibilité
Si l'extension grpc est installée mais que les variables d'environnement correspondantes n'ont pas été définies, un processus de montage supplémentaire sera créé lors du démarrage, ce qui entraînera un échec de l'arrêt.

**Solution :**
