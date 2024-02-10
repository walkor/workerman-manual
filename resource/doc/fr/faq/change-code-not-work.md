# Changements non pris en compte après modification du code

**Raison :**

Workerman est un programme résidant en mémoire, ce qui permet d'éviter la lecture répétée du disque et la recompilation du code PHP à chaque fois, afin d'obtenir les meilleures performances. Ainsi, après avoir modifié le code métier, il est nécessaire de recharger manuellement ou de redémarrer pour que les modifications prennent effet.

Par ailleurs, Workerman fournit un service de surveillance des mises à jour de fichiers. Ce service détecte les mises à jour de fichiers et relance automatiquement le programme en rechargeant les fichiers PHP. Il suffit d'intégrer ce service au projet et de le démarrer en même temps que le projet.

Remarque : le rechargement n'est pas pris en charge sur les systèmes Windows et le service de surveillance n'est pas utilisable.


**Adresse de téléchargement du service de surveillance des fichiers :**

1. Version sans dépendance : https://github.com/walkor/workerman-filemonitor

2. Version utilisant inotify : https://github.com/walkor/workerman-filemonitor-inotify

**Différences entre les deux versions :**

La version à l'adresse 1 utilise une méthode de vérification périodique de la dernière modification des fichiers, tandis que la version à l'adresse 2 utilise le mécanisme [inotify](https://baike.baidu.com/view/2645027.htm) du noyau Linux, qui notifie le système lors des mises à jour de fichiers.

En général, la version sans dépendance à l'adresse 1 est suffisante.
