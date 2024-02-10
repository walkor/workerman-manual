# Concentration des requêtes dans certains processus

### Phénomène
Parfois, lorsque nous utilisons la commande `php start.php status`, nous remarquons que les requêtes sont concentrées dans certains processus spécifiques, tandis que d'autres processus restent complètement inactifs.

### Mécanisme de préemption
Par défaut, Workerman utilise un mécanisme de préemption pour que plusieurs processus puissent obtenir des connexions. Cela signifie que lorsque le client établit une connexion, tous les processus inactifs ont l'opportunité de la récupérer, le plus rapide l'emporte. La rapidité est déterminée par la planification du noyau du système d'exploitation. Celui-ci peut favoriser le processus utilisé le plus récemment pour l'utilisation du processeur, car les informations de contexte du processus précédent peuvent encore être présentes dans les registres du processeur, ce qui permet de réduire les coûts de changement de contexte. Ainsi, lorsque les performances des opérations sont suffisamment rapides ou lors de tests de charge, il est plus probable que les connexions soient traitées par certains processus, car cette stratégie permet d'éviter des changements fréquents de processus et offre généralement des performances optimales.

### Mécanisme de rotation
En configurant `$worker->reusePort = true;`, Workerman peut changer le mécanisme d'obtention des connexions pour une rotation, où le noyau répartit les connexions de manière approximativement égale entre tous les processus. Ainsi, tous les processus traitent les requêtes ensemble.

### Idée fausse
Beaucoup de développeurs pensent qu'une meilleure performance est assurée lorsque tous les processus participent au traitement des requêtes, mais ce n'est pas toujours le cas. Lorsque l'opération est suffisamment simple, le nombre de processus impliqués dans le traitement des requêtes se rapprochant du nombre de cœurs de CPU permet d'atteindre le plus haut débit sur le serveur. Par exemple, un serveur quadricœur avec 4 processus configurés affiche généralement le QPS le plus élevé lors des tests de charge d'une simple page "helloworld". Si le nombre de processus impliqués dans le traitement dépasse largement le nombre de cœurs de CPU, les coûts de contexte des processus augmentent, ce qui entraîne des performances plus faibles. En général, pour des activités impliquant des bases de données, le fait de configurer de 3 à 6 fois le nombre de cœurs de CPU comme le nombre de processus peut améliorer les performances.
