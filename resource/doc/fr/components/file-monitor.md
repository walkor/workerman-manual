# Composant de surveillance de fichiers

**Contexte :**

Workerman fonctionne en mémoire résidente, ce qui permet d'éviter de relire à plusieurs reprises le disque et de réinterpréter le code PHP afin d'atteindre des performances maximales. Ainsi, après avoir modifié le code métier, il est nécessaire de recharger manuellement ou de redémarrer pour que les modifications prennent effet.

En même temps, Workerman fournit un service de surveillance des mises à jour de fichiers. Ce service détecte les mises à jour de fichiers et exécute automatiquement le rechargement, ce qui permet de recharger les fichiers PHP. Les développeurs peuvent l'intégrer dans leur projet et il se lance en même temps que le projet.

**Adresse de téléchargement du service de surveillance de fichiers :**

1. Version sans dépendance : https://github.com/walkor/workerman-filemonitor

2. Version dépendante d'inotify : https://github.com/walkor/workerman-filemonitor-inotify (nécessite l'installation de l'extension [inotify](https://php.net/manual/zh/book.inotify.php))

**Différences entre les deux versions :**

La version à l'adresse 1 utilise une méthode de sondage de l'heure de mise à jour des fichiers chaque seconde pour déterminer si le fichier a été mis à jour.

La version à l'adresse 2 utilise le mécanisme du noyau Linux [inotify](https://baike.baidu.com/view/2645027.htm), de sorte que le système notifie directement Workerman lorsqu'un fichier est mis à jour.

En général, la première version sans dépendance est suffisante.

**Mode d'emploi :**

Copiez le répertoire Applications/FileMonitor dans le répertoire Applications de votre projet.

Si votre projet n'a pas de répertoire Applications, vous pouvez copier le fichier Applications/FileMonitor/start.php dans n'importe quel emplacement de votre projet et le requérir dans le script de démarrage.

Le composant de surveillance surveille par défaut le répertoire Applications. Si vous avez besoin de le modifier, modifiez la variable `$monitor_dir` dans Applications/FileMonitor/start.php. Il est recommandé d'utiliser un chemin absolu pour la valeur de `$monitor_dir`.

**Remarques :**

* Le système Windows ne prend pas en charge le rechargement, donc ce service de surveillance ne peut pas être utilisé.
* Il ne fonctionne que en mode débogage. Il ne fonctionnera pas en mode daemon (voir explications ci-dessous).
* Seuls les fichiers chargés après l'exécution de Worker::runAll peuvent être mis à jour à chaud, ou bien seuls les fichiers chargés dans les rappels onXXX peuvent être mis à jour à chaud.

**Pourquoi ne prend-il pas en charge le mode daemon ?**

Le mode daemon est généralement utilisé pour l'exécution en production. Lors de la publication en production, plusieurs fichiers sont généralement publiés simultanément, et il peut y avoir des dépendances entre les fichiers. Comme la synchronisation de plusieurs fichiers sur le disque nécessite un certain temps, il peut arriver qu'à un moment donné tous les fichiers ne soient pas présents sur le disque. Si une mise à jour de fichier est détectée à ce moment-là et qu'un rechargement est exécuté, cela peut entraîner des erreurs fatales dues à des fichiers manquants.

De plus, en production, il arrive parfois que des bogues soient corrigés en ligne. Si vous modifiez directement le code et que les modifications prennent immédiatement effet, cela peut entraîner une erreur de syntaxe rendant le service en ligne indisponible. La bonne méthode consiste à enregistrer le code, puis à vérifier s'il y a des erreurs de syntaxe avec `php -l yourfile.php`, avant de recharger le code mis à jour.

Si un développeur a vraiment besoin d'activer la surveillance des fichiers et la mise à jour automatique en mode daemon, il peut modifier le code de Applications/FileMonitor/start.php en supprimant la vérification de Worker::$daemonize.
