# Caractéristiques de Workerman

### 1. Développement pur en PHP
Les applications développées avec Workerman peuvent fonctionner de manière indépendante sans dépendre de conteneurs tels que php-fpm, apache ou nginx. Cela simplifie grandement le développement, le déploiement et le débogage des applications pour les développeurs PHP.

### 2. Support des processus multiples en PHP
Pour exploiter pleinement les performances des serveurs multi-CPU, Workerman prend en charge par défaut les tâches multi-processus. Workerman lance un processus principal et plusieurs sous-processus pour fournir des services. Le processus principal surveille les sous-processus, qui, quant à eux, gèrent individuellement la connexion réseau, la réception, le traitement et l'envoi des données. Grâce à ce modèle de processus simple, Workerman offre une plus grande stabilité et une efficacité accrue.

### 3. Support de TCP et UDP
Workerman prend en charge deux protocoles de transport, TCP et UDP. Changer de protocole de transport ne nécessite qu'une modification d'attribut, sans modifier le code métier.

### 4. Support des connexions persistantes
Dans de nombreuses situations, une application PHP doit maintenir des connexions persistantes avec les clients, comme dans le cas des salons de discussion, des jeux, etc. Cependant, les conteneurs PHP traditionnels (apache, nginx, php-fpm) ont du mal à gérer cela. Avec Workerman, tant que le service côté serveur n'appelle pas activement l'API de fermeture de la connexion, les connexions persistantes en PHP peuvent être utilisées. Un seul processus Workerman peut prendre en charge des dizaines de milliers de connexions concurrentes, tandis que plusieurs processus peuvent prendre en charge des centaines de milliers, voire des millions de connexions concurrentes.

### 5. Support de divers protocoles de niveau applicatif
L'interface de Workerman prend en charge différents protocoles de niveau applicatif, y compris les protocoles personnalisés. Changer de protocole dans Workerman est également très simple ; il suffit de configurer un champ, le protocole change automatiquement, sans modifier le code métier. Il est même possible de démarrer plusieurs ports avec des protocoles différents pour répondre aux besoins des différents clients.

### 6. Support de la haute concurrence
Workerman prend en charge la bibliothèque de bouclage d'événements Libevent (nécessitant l'installation de l'extension event). L'utilisation de l'événement pour une haute concurrence avec des connexions persistantes est particulièrement performante. En l'absence de l'extension Event, PHP utilise les appels système de sélection intégrés, offrant des performances tout aussi puissantes.

### 7. Support du redémarrage en douceur des services
Lorsqu'un redémarrage du service est nécessaire (par exemple lors de la publication d'une nouvelle version), il est préférable que les processus qui traitent actuellement les demandes des utilisateurs ne soient pas immédiatement arrêtés, et encore moins que le redémarrage n'entrave les communications avec les clients. Workerman propose une fonctionnalité de redémarrage en douceur, garantissant une mise à niveau sans heurts du service, sans affecter l'utilisation côté client.

### 8. Support de la détection et du chargement automatique des fichiers mis à jour
Pendant le développement, il est essentiel que les modifications apportées au code prennent effet immédiatement. Workerman propose le [composant de surveillance des fichiers (FileMonitor)](../components/file-monitor.md). Dès qu'un fichier est mis à jour, Workerman s'exécute automatiquement pour charger la nouvelle version, permettant ainsi de voir les résultats instantanément.

### 9. Possibilité de définir l'utilisateur exécutant les sous-processus
Puisque les sous-processus sont les processus qui gèrent effectivement les requêtes des utilisateurs, il est important, pour des raisons de sécurité, de ne pas leur accorder des privilèges trop élevés. C'est pourquoi Workerman permet de définir l'utilisateur exécutant les sous-processus, renforçant ainsi la sécurité du serveur.

### 10. Prise en charge de la rétention permanente d'objets ou de ressources
Pendant le fonctionnement, Workerman ne charge et n'analyse les fichiers PHP qu'une seule fois, les maintenant en mémoire. Ainsi, les déclarations de classes et de fonctions, l'environnement d'exécution PHP, les tables de symboles, etc., ne sont pas constamment créés et supprimés, ce qui diffère totalement du fonctionnement de PHP dans un conteneur Web. Dans Workerman, les membres statiques ou les variables globales d'un processus sont conservés de manière permanente tout au long du cycle de vie du processus. Ainsi, un objet ou une ressource placé dans une variable globale ou un membre statique de classe peut être réutilisé pour toutes les demandes de ce processus, évitant ainsi de créer constamment et de détruire des connexions à la base de données. Cela permet d'économiser le temps nécessaire aux trois poignées de main TCP lors de la connexion à une base de données, à l'authentification de l'utilisateur, et aux quatre poignées de main TCP lors de la déconnexion, ce qui améliore considérablement l'efficacité de l'application.

### 11. Haute performance
Étant donné que les fichiers PHP sont chargés à partir du disque, analysés une seule fois, puis conservés en mémoire, la prochaine fois qu'ils sont utilisés, les opcodes en mémoire sont directement utilisés, réduisant considérablement les E/S sur disque et le temps nécessaire à l'initialisation d'une demande dans PHP, à la création de l'environnement d'exécution, à l'analyse lexicale, à l'analyse syntaxique, à la compilation d'opcodes, à la fermeture de la demande, etc. De plus, en l'absence de dépendance à nginx, apache ou tout autre conteneur, les coûts de communication entre Workerman et PHP sont réduits. Le fait que les ressources puissent être conservées en permanence et ne nécessitent pas une initialisation constante, comme pour une connexion à la base de données, confère à Workerman un très haut niveau de performance pour le développement d'applications.

### 12. Prise en charge de HHVM
Workerman peut fonctionner sur la machine virtuelle HHVM, offrant des performances PHP considérablement améliorées. En particulier pour les tâches nécessitant une forte puissance de calcul, les performances sont excellentes. Après des tests de charge réels comparatifs, Workerman en cours d'exécution sur HHVM, dans des conditions de faible charge, a augmenté le débit du réseau de 30 à 80% par rapport à l'exécution sur Zend PHP 5.6.

### 13. Support du déploiement distribué

### 14. Support du mode daemon

### 15. Support de l'écoute sur plusieurs ports

### 16. Prise en charge de la redirection de l'entrée et de la sortie standard
