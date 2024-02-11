# Échec de démarrage de workerman

## Phénomène 1
Après le démarrage, une erreur similaire est signalée :
```php
php start.php start
PHP Warning:  stream_socket_server() : unable to connect to tcp://xx.xx.xx.xx:xxxx (Address already in use) in ...workerman/Worker.php on line xxxx
```
**Mots-clés** : ```Address already in use```

**Cause fondamentale** : Le port est déjà utilisé et ne peut pas être démarré.

#### Solution 1

Vous pouvez utiliser la commande ```netstat -anp | grep numéro de port``` pour savoir quel programme utilise le port.
Ensuite, arrêtez le programme correspondant pour libérer le port.

#### Solution 2
Si vous ne pouvez pas arrêter le programme correspondant au port, vous pouvez résoudre ce problème en changeant le port de workerman.

#### Solution 3
Si c'est le port utilisé par Workerman et qu'il ne peut pas être arrêté par la commande stop (généralement en raison de la perte du fichier pid ou du fait que le processus principal a été tué par le développeur), vous pouvez tuer le processus Workerman en exécutant les deux commandes suivantes.

```shell
killall php
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

#### Solution 4
S'il n'y a vraiment aucun programme écoutant ce port, le développeur peut avoir configuré deux ou plusieurs écoutes dans workerman avec le même port. Veuillez vérifier si le script de démarrage écoute le même port.

#### Solution 5
Vérifiez si le programme a activé le "reusePort" et désactivez-le pour essayer.

## Phénomène 2
Après le démarrage, une erreur similaire est signalée :
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Cannot assign requested address) in ...workerman/Worker.php on line xxxx
```
ou
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (dans son contexte, l'adresse demandée est invalide) in ...workerman/Worker.php on line xxxx
```
**Mots-clés** : ``Cannot assign requested address`` ou ``dans son contexte, l'adresse demandée est invalide``

**Raison de l'échec** :

Le paramètre IP écouté dans le script de démarrage est incorrect ou n'est pas l'IP de la machine locale. Veuillez entrer l'IP de la machine locale ou utilisez ```0.0.0.0``` (pour écouter toutes les adresses IP locales) pour résoudre ce problème.

**Conseil** : Sur un système Linux, vous pouvez utiliser la commande ```ifconfig``` pour voir toutes les adresses IP de la machine locale.
Si vous êtes un utilisateur de serveur cloud (Alibaba Cloud/Tencent Cloud, etc.), veuillez noter que votre adresse IP publique est en fait une adresse IP privée (par exemple, le réseau privé Alibaba Cloud) et que l'adresse IP publique ne fait pas partie du serveur actuel. Par conséquent, il ne peut pas être écouté par l'adresse IP publique. Vous pouvez toujours lier ```0.0.0.0``` même si vous ne pouvez pas utiliser l'adresse publique.

## Phénomène 3
```php
Warning: stream_socket_server has been disabled for security reasons in ...
```
**Raison de l'échec** :

La fonction stream_socket_server est désactivée dans le fichier php.ini.

**Solution**

1. Exécutez ```php --ini``` pour trouver le fichier php.ini.

2. Ouvrez php.ini, trouvez la ligne disable_functions, supprimez l'entrée de l'interdiction de stream_socket_server.

## Phénomène 4
```php
PHP Warning: stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permission denied)
```
**Cause de l'échec** :

Sous Linux, si le port écouté est inférieur à 1024, un accès root est nécessaire.

**Solution**

Utilisez un port supérieur à 1024 ou démarrez le service en tant qu'utilisateur root.
