# Désactivation de la vérification des fonctions

Utilisez ce script pour vérifier si des fonctions sont désactivées. Exécutez la commande suivante dans votre terminal : ```curl -Ss https://www.workerman.net/check | php```

Si vous voyez s'afficher le message suivant : ```La fonction nom_de_la_fonction est peut-être désactivée. Veuillez vérifier les disable_functions dans php.ini```, cela signifie que les fonctions sur lesquelles dépend workerman sont désactivées. Vous devez lever cette désactivation dans php.ini pour pouvoir utiliser workerman normalement.
Pour lever la désactivation, suivez l'une des deux méthodes suivantes au choix.

## Méthode 1 : Levée de la désactivation via un script

Exécutez le script suivant `curl -Ss https://www.workerman.net/fix | php` pour lever la désactivation.

## Méthode 2 : Levée de la désactivation manuellement

**Voici les étapes à suivre :**

1. Exécutez `php --ini` pour trouver l'emplacement du fichier php.ini utilisé par php cli.

2. Ouvrez php.ini et trouvez l'entrée `disable_functions` pour lever la désactivation des fonctions correspondantes.

**Fonctions requises**
Pour utiliser workerman, vous devez lever la désactivation des fonctions suivantes :
```stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
