# Optimisation du noyau Linux

Pour que le système puisse supporter un plus grand nombre de connexions simultanées, en plus de l'installation obligatoire de l'extension [event] (../install/install.md), l'optimisation du noyau Linux est également **d'une importance capitale**. Chaque optimisation **ci-dessous** est extrêmement importante, veuillez vous assurer de les effectuer une par une.

**Explications des paramètres:**

> **max-file**: Représente la quantité de poignées de fichiers pouvant être ouvertes au niveau **système**. Ceci s'applique à l'ensemble du système d'exploitation, et non aux utilisateurs individuels.
> 
> **ulimit -n**: Contrôle la quantité de poignées de fichiers pouvant être ouvertes au niveau du **processus**. Cela contrôle le nombre de poignées de fichiers disponibles pour l'utilisateur courant du `shell` et pour les processus qu'il lance.

Pour vérifier la quantité de poignées de fichiers pouvant être ouvertes au niveau **système**: `cat /proc/sys/fs/file-max`

Ouvrez le fichier /etc/sysctl.conf, et ajoutez les réglages suivants:
```conf
# Ce paramètre définit la quantité de TIME_WAIT du système. Si elle dépasse la valeur par défaut, elle sera immédiatement effacée
net.ipv4.tcp_max_tw_buckets = 20000
# Définit la longueur maximale de la file d'attente pour chaque port dans le système. Ceci est un paramètre global
net.core.somaxconn = 65535
# Le maximum de backlog des demandes de connexion qui n'ont pas encore reçu de confirmation de la part du destinataire
net.ipv4.tcp_max_syn_backlog = 262144
# Lorsque le débit de paquets entrants sur une interface réseau dépasse le débit auquel le noyau les traite, le nombre maximal de paquets à mettre dans la file d'attente est autorisé
net.core.netdev_max_backlog = 30000
# Cette option entraînera des délais d'attente pour les clients dans un réseau NAT. Il est recommandé de laisser la valeur à 0. Depuis le noyau 4.12, Linux a supprimé la configuration de tcp_tw_recycle. Si vous obtenez une erreur "Aucun fichier ou dossier de ce type", veuillez l'ignorer
net.ipv4.tcp_tw_recycle = 0
# Nombre total de fichiers que tous les processus du système peuvent ouvrir
fs.file-max = 6815744
# Taille de la table de suivi du pare-feu. Remarque : si le pare-feu n'est pas activé, une erreur "net.netfilter.nf_conntrack_max" est une clé inconnue peut apparaître. Vous pouvez l'ignorer
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```
Exécutez `sysctl -p` pour que les modifications prennent effet immédiatement.

**Remarque:**

/etc/sysctl.conf offre de nombreuses options de configuration. D'autres options peuvent être configurées en fonction de vos besoins environnementaux.

## Nombre de fichiers ouverts

Réglez le nombre de fichiers ouverts par système pour résoudre le problème de ```trop de fichiers ouverts``` dans des situations de forte charge. Cette option affecte directement le nombre maximum de connexions clients qu'un processus individuel peut gérer.

Les fichiers ouverts en douceur (`Soft open files`) sont des paramètres du système Linux qui influent sur le nombre maximal de poignées de fichiers qu'un processus unique peut ouvrir. Cette valeur affectera le nombre de connexions utilisateur qu'un processus unique peut conserver dans des applications à connexions persistantes telles que la messagerie. Vous pouvez voir cette valeur en exécutant `ulimit -n`. Si elle est de 1024, cela signifie qu'un processus unique ne pourra gérer simultanément que 1024 connexions, voire moins (car d'autres poignées de fichiers sont ouvertes). Si vous lancez 4 processus pour gérer les connexions utilisateur, l'application ne pourra pas prendre en charge plus de 4*1024 connexions simultanées, ce qui signifie qu'elle peut prendre en charge un maximum de 4x1024 utilisateurs en ligne. Vous pouvez augmenter ce réglage pour permettre au service de prendre en charge un plus grand nombre de connexions TCP.

**Trois méthodes pour modifier les fichiers ouverts en douceur (`Soft open files`)**:

Première méthode : Exécutez directement `ulimit -HSn 102400` dans le terminal, puis redémarrez workerman.

Cela s'applique uniquement au terminal actuel. Après l'avoir quitté, le nombre de fichiers ouverts reviendra à la valeur par défaut.

Deuxième méthode : Ajoutez `ulimit -HSn 102400` à la fin du fichier `/etc/profile`. Ainsi, il sera exécuté automatiquement chaque fois que vous vous connecterez. Après la modification, redémarrez workerman.

Troisième méthode : Pour que la modification du nombre de fichiers ouverts soit permanente, vous devez modifier le fichier de configuration : `/etc/security/limits.conf`. Ajoutez ceci à la fin du fichier :

```
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

Cette méthode nécessite un redémarrage du serveur pour prendre effet.
