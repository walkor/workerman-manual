# Installation d'extensions
## Remarque
Contrairement au mode d'exécution d'Apache+PHP ou Nginx+PHP, WorkerMan s'exécute sur PHP en ligne de commande [PHP CLI](http://php.net/manual/zh/features.commandline.php), utilisant un exécutable PHP différent et peut avoir un fichier php.ini différent. Ainsi, afficher ```phpinfo()``` dans une page Web pour vérifier l'installation d'une extension ne signifie pas que l'extension correspondante est également installée pour PHP CLI en ligne de commande.

## Comment vérifier quelles extensions sont installées pour PHP CLI
Exécutez la commande ```php -m``` pour lister les extensions déjà installées pour PHP CLI. Le résultat sera similaire à ceci :
```shell
~# php -m
[PHP Modules]
event
posix
pcntl
...
```

## Comment déterminer l'emplacement du fichier php.ini pour PHP CLI
Lors de l'installation d'extensions, il peut être nécessaire de configurer manuellement le fichier php.ini en y ajoutant des extensions. Pour cela, l'emplacement du fichier php.ini pour PHP CLI doit être confirmé. Vous pouvez exécuter ```php --ini``` pour rechercher l'emplacement du fichier php.ini pour PHP CLI, le résultat sera similaire à ce qui suit (les résultats affichés varieront selon les systèmes) :
```shell
~# php --ini
Configuration File (php.ini) Path: /etc/php8/cli
Loaded Configuration File:         /etc/php8/cli/php.ini
Scan for additional .ini files in: /etc/php8/cli/conf.d
Additional .ini files parsed:      /etc/php8/cli/conf.d/apc.ini,
/etc/php8/cli/conf.d/pdo.ini,
/etc/php8/cli/conf.d/pdo_mysql.ini
...
```

# Installation d'une extension pour PHP CLI (installation de l'extension memcached)
## Méthode 1 : Utiliser apt ou yum pour l'installation
Si PHP a été installé via la commande apt ou yum, les extensions peuvent également être installées via apt ou yum.

**Méthode pour l'installation de l'extension sur les systèmes debian/ubuntu avec apt (les utilisateurs non root doivent ajouter la commande sudo)**

1. Utiliser ```apt-cache search``` pour rechercher le paquet d'extension
```shell
~# apt-cache search memcached php
php-apc - module APC (Alternative PHP Cache) pour PHP 5
php5-memcached - module memcached pour php5
```
2. Utiliser ```apt-get install``` pour installer le paquet d'extension
```shell
~# apt-get install -y php5-memcached
Reading package lists... Done
Reading state information... Done
...
```

**Méthode pour l'installation de l'extension sur les systèmes centos avec yum**

1. Utiliser ```yum search``` pour rechercher le paquet d'extension
```shell
~# yum search memcached php
php-pecl-memcached - module memcached pour php5
```
2. Utiliser ```yum install``` pour installer le paquet d'extension
```shell
~# yum install -y php-pecl-memcached
Reading package lists... Done
Reading state information... Done
...
```
**Remarque:**

L'installation des extensions PHP via apt ou yum configure automatiquement le fichier php.ini, rendant les extensions utilisables immédiatement. Cependant, certaines extensions peuvent ne pas avoir de paquet d'installation correspondant dans apt ou yum.

## Méthode 2 : Utilisation de pecl pour l'installation
Utilisez la commande ```pecl install``` pour installer une extension

1. Installation via ```pecl install```
```shell
~# pecl install memcached
downloading memcached-2.2.0.tgz ...
Starting to download memcached-2.2.0.tgz (70,449 bytes)
....
```
2. Configurer php.ini

Recherchez l'emplacement du fichier php.ini en exécutant ```php --ini```, puis ajoutez ```extension=memcached.so``` dans le fichier.

## Méthode 3 : Installation via la compilation à partir du code source (généralement pour installer des extensions fournies avec PHP, en utilisant l'extension pcntl comme exemple)
1. Utilisez la commande ```php -v``` pour vérifier la version actuelle de PHP CLI
```shell
~# php -v
PHP 5.3.29-1~dotdeb.0 with Suhosin-Patch (cli) (built: Aug 14 2014 19:55:20)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2014 Zend Technologies
```
2. Téléchargez le code source de PHP en fonction de la version

Page de téléchargement des versions historiques de PHP: https://php.net/releases/

3. Décompressez le fichier source

Par exemple, si le fichier téléchargé est nommé ```php-5.3.29.tar.gz``` :
```shell
~# tar -zxvf php-5.3.29.tar.gz
php-5.3.29/
php-5.3.29/README.WIN32-BUILD-SYSTEM
php-5.3.29/netware/
...
```
4. Accédez au répertoire ext/pcntl du code source
```shell
~# cd php-5.3.29/ext/pcntl/
```
5. Exécutez la commande ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```
6. Exécutez la commande ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
...
```
7. Exécutez la commande ```make```
```shell
~# make
/bin/bash /tmp/php-5.3.29/ext/pcntl/libtool --mode=compile cc ...
-I/usr/include/php5 -I/usr/include/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend...
...
```
8. Exécutez la commande ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```
9. Configurez le fichier ini

Recherchez l'emplacement du fichier php.ini en exécutant ```php --ini```, puis ajoutez ```extension=pcntl.so``` dans le fichier.

**Remarque :**
Cette méthode est généralement utilisée pour installer des extensions fournies avec PHP, telles que les extensions posix et pcntl. En plus d'utiliser phpize pour compiler une extension, il est également possible de recompiler PHP complet lors du paramétrage des extensions, par exemple, exécuter les commandes suivantes dans le répertoire racine du code source :
```shell
~# ./configure --enable-pcntl --enable-posix ...
~# make && make install
```

## Méthode 4 : Installation via phpize
Si vous devez installer une extension qui n'est pas présente dans le répertoire ext du code source PHP, cette extension doit être recherchée et téléchargée depuis https://pecl.php.net 

Pour installer l'extension libevent (supposant que la bibliothèque libevent-dev est installée sur le système) :

1. Téléchargez le fichier compressé de l'extension libevent (à partir de n'importe quel répertoire sur le système actuel)
```shell
~# wget https://pecl.php.net/get/libevent-0.1.0.tgz
--2015-05-26 21:43:40--  https://pecl.php.net/get/libevent-0.1.0.tgz
Resolving pecl.php.net... 104.236.228.160
Connecting to pecl.php.net|104.236.228.160|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9806 (9.6K) [application/octet-stream]
Saving to: “libevent-0.1.0.tgz”

100%[=======================================================>] 9,806       41.4K/s   in 0.2s

```

2. Décompressez le fichier de l'extension
```shell
~# tar -zxvf libevent-0.1.0.tgz
package.xml
libevent-0.1.0/config.m4
libevent-0.1.0/CREDITS
libevent-0.1.0/libevent.c
....
```

3. Accédez au répertoire source
```shell
~# cd libevent-0.1.0/
```

4. Exécutez la commande ```phpize```
```shell
~# phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
```

5. Exécutez la commande ```configure```
```shell
~# ./configure
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for a sed that does not truncate output... /bin/sed
checking for cc... cc
checking whether the C compiler works... yes
...
```

6. Exécutez la commande ```make```
```shell
~# /bin/bash /data/test/libevent-0.1.0/libtool --mode=compile cc  -I. -I/data/test/libevent-0.1.0 -DPHP_ATOM_INC -I/data/test/libevent-0.1.0/include
...
```

7. Exécutez la commande ```make install```
```shell
~# make install
Installing shared extensions:     /usr/lib/php5/20090626/
```

8. Configurez le fichier ini

Recherchez l'emplacement du fichier php.ini en exécutant ```php --ini```, puis ajoutez ```extension=libevent.so``` dans le fichier.
