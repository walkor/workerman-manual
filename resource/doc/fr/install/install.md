# Instructions d'installation
Workerman est en fait un package de code PHP. Si votre environnement PHP est déjà installé, il suffit de télécharger le code source ou la démo de Workerman pour le faire fonctionner.

**Installation avec Composer :**
```sh
composer require workerman/workerman
```

> **Remarque :**
> Certains miroirs de proxy Composer ne sont pas complets. Utilisez la commande ci-dessus `composer config -g --unset repos.packagist` pour supprimer le proxy.

# Utilisateurs de Windows (lecture obligatoire)
À partir de la version 3.5.3 de Workerman, ce dernier est compatible avec les systèmes Windows et Linux.
Les utilisateurs Windows doivent configurer les variables d'environnement PHP.

 ` ===Les instructions ci-dessous s'appliquent uniquement à l'environnement Linux de Workerman, veuillez ignorer si vous êtes un utilisateur Windows=== `

# Vérification de l'environnement du système Linux
Le script suivant peut être utilisé sur un système Linux pour tester si votre environnement PHP est conforme aux exigences d'exécution de Workerman.
 `curl -Ss https://www.workerman.net/check | php`

Si toutes les vérifications du script ci-dessus sont correctes, cela signifie que votre environnement PHP est conforme aux exigences de Workerman, et vous pouvez télécharger l'exemple directement depuis le [site officiel](https://www.workerman.net/) pour l'exécuter.

Si toutes les vérifications ne sont pas correctes, veuillez vous référer à la documentation ci-dessous pour installer les extensions manquantes.

(Note : le script de vérification n'inclut pas l'extension événementielle. Si le nombre de connexions concurrentes dans l'entreprise dépasse 1024, vous devez installer l'extension événementielle et optimiser le noyau Linux, veuillez vous référer aux instructions ci-dessous pour installer l'extension manquante)

# Installation des extensions manquantes pour un environnement PHP existant

## Installation des extensions pcntl et posix :

**Systèmes CentOS**
Si PHP est installé via yum, exécutez la commande suivante dans votre terminal :
```shell
yum install php-process
```
Cela installera les extensions pcntl et posix.

Si l'installation échoue ou si PHP n'a pas été installé avec yum, veuillez vous référer à la section "Compiler et installer à partir du code source" dans le manuel [Annexe - Installation d'extensions](../appendices/install-extension.md).

**Systèmes Debian/Ubuntu/macOS**
Veuillez vous référer à la section "Compiler et installer à partir du code source" dans le manuel [Annexe - Installation d'extensions](../appendices/install-extension.md).

## Installation de l'extension Event :
Pour supporter un plus grand nombre de connexions concurrentes, il est nécessaire d'installer l'extension Event et d'optimiser le noyau Linux. Voici la méthode d'installation :

**Systèmes CentOS**

1. Installation du paquet libevent-devel nécessaire à l'extension Event, exécutez la commande suivante dans votre terminal :
```shell
yum install libevent-devel -y
# Si l'installation échoue, essayez la commande suivante
# yum install libevent2-devel -y
```

2. Installation de l'extension Event, exécutez la commande suivante dans votre terminal :
(L'extension Event nécessite PHP>=5.4)
```shell
pecl install event
```
Lorsqu'il est demandé : ```Include libevent OpenSSL support [yes] :```, entrez ```no``` et appuyez sur Entrée, puis appuyez sur Entrée pour les autres options.

3. Exécutez la commande ```php --ini```, trouvez et ouvrez le fichier php.ini, et ajoutez la configuration suivante à la dernière ligne :
```shell
extension=event.so
```

**Systèmes Debian/Ubuntu**

1. Installation du paquet libevent-dev nécessaire à l'extension Event, exécutez la commande suivante dans votre terminal :
```shell
apt-get install libevent-dev -y
# Si l'installation échoue, veuillez essayer la commande suivante
# apt-get install libevent2-dev -y
```

2. Installation de l'extension Event, exécutez la commande suivante dans votre terminal :
```shell
pecl install event
```
Lorsqu'il est demandé : ```Include libevent OpenSSL support [yes] :```, entrez ```no``` and appuyez sur Entrée, puis appuyez sur Entrée pour les autres options.

3. Exécutez la commande ```php --ini```, trouvez et ouvrez le fichier php.ini, et ajoutez la configuration suivante à la dernière ligne :
```shell
extension=event.so
```

**Procédure d'installation pour le système macOS**
Le système macOS est généralement utilisé comme machine de développement, il n'est donc pas nécessaire d'installer l'extension Event.

# Installation d'un nouveau système (Installation complète de PHP + extensions)

## Procédure d'installation pour le système CentOS

1. Exécutez la commande suivante dans votre terminal (cette étape inclut l'installation du programme principal php-cli, des extensions pcntl, posix, de la bibliothèque libevent et du programme git) :
```shell
yum install php-cli php-process git gcc php-devel php-pear libevent-devel -y
```

2. Installez l'extension Event en exécutant la commande suivante dans votre terminal :
(Remarque : l'extension Event nécessite PHP>=5.4)
```shell
pecl install event
```
Lorsqu'il est demandé : ```Include libevent OpenSSL support [yes] :```, entrez ```no``` et appuyez sur Entrée, puis appuyez sur Entrée pour les autres options.

3. Exécutez la commande ```php --ini```, trouvez et ouvrez le fichier php.ini, et ajoutez la configuration suivante à la dernière ligne :
```shell
extension=event.so
```

4. Exécutez la commande suivante dans votre terminal (cette étape télécharge le programme principal de Workerman à partir de Github) :
```shell
git clone https://github.com/walkor/Workerman
```

5. Consultez la section "Exemple de développement simple" dans [Guide de démarrage rapide](../getting-started/simple-example.md) pour écrire le fichier d'entrée et l'exécuter, ou téléchargez la démo à partir du [site officiel](https://www.workerman.net/) pour l'exécuter.

## Procédure d'installation pour le système Debian/Ubuntu

1. Exécutez la commande suivante dans votre terminal (cette étape inclut l'installation du programme principal php-cli, de la bibliothèque libevent, et du programme git) :
```shell
apt-get install php-cli git gcc php-pear php-dev libevent-dev -y
```

2. Installez l'extension Event en exécutant la commande suivante dans votre terminal :
(Remarque : l'extension Event nécessite PHP>=5.4)
```shell
pecl install event
```
Lorsqu'il est demandé : ```Include libevent OpenSSL support [yes] :```, entrez ```no``` et appuyez sur Entrée, puis appuyez sur Entrée pour les autres options.

3. Exécutez la commande ```php --ini```, trouvez et ouvrez le fichier php.ini, et ajoutez la configuration suivante à la dernière ligne :
```shell
extension=event.so
```

4. Exécutez la commande suivante dans votre terminal (cette étape télécharge le programme principal de Workerman à partir de Github) :
```shell
git clone https://github.com/walkor/Workerman
```

5. Consultez la section "Exemple de développement simple" dans [Guide de démarrage rapide](../getting-started/simple-example.md) pour écrire le fichier d'entrée et l'exécuter, ou téléchargez la démo à partir du [site officiel](https://www.workerman.net/) pour l'exécuter.

## Procédure d'installation pour le système macOS
**Méthode 1 :** macOS est livré avec PHP Cli, mais il peut manquer l'extension ```pcntl```.

1. Veuillez vous référer à la section "Compilation et installation à partir du code source" dans le manuel [Annexe - Installation d'extensions](../appendices/install-extension.md) pour installer l'extension ```pcntl```.

2. Veuillez vous référer à la section "Utilisation de phpize pour installer l'extension ```event```" dans le manuel [Annexe - Installation d'extensions](../appendices/install-extension.md) (Ceci peut être omis pour une machine de développement).

3. Téléchargez le programme principal de Workerman à partir de [ce lien](https://www.workerman.net/download/workermanzip) ou depuis le [site officiel](https://www.workerman.net/) pour l'exécuter.

**Méthode 2 :** Installation de PHP et de ses extensions correspondantes avec la commande ```brew```.

1. Exécutez la commande suivante dans votre terminal pour installer l'outil ```brew``` (si vous avez déjà installé ```brew```, vous pouvez passer cette étape) :
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. Exécutez la commande suivante dans votre terminal pour installer ```php``` :
```shell
brew install php
```

3. Exécutez la commande suivante dans votre terminal pour installer l'extension ```event``` :
```shell
brew install php-event
```

4. Téléchargez la démo à partir du [site officiel](https://www.workerman.net/) pour l'exécuter.

# Information sur l'extension Event
L'installation de l'extension [Event](https://php.net/manual/zh/book.event.php) n'est pas obligatoire. L'extension est recommandée pour soutenir un grand nombre de connexions concurrentes supérieur à 1000. Si votre entreprise n'a pas un nombre élevé de connexions concurrentes, comme moins de 1000 connexions, vous n'avez pas besoin de l'installer.

## Problèmes courants
1. Si vous rencontrez l'erreur suivante :  `checking for include/event2/event.h... not found`, veuillez essayer de supprimer le paquet libevent-dev(el) et installer libevent2-dev(el) à la place.
Pour les systèmes CentOS : yum remove libevent-devel && yum install libevent2-devel
Pour les systèmes Debian/Ubuntu : apt-get remove libevent-dev && apt-get install libevent2-dev

2. Si vous rencontrez l'erreur suivante : `NOTICE: PHP message: PHP Warning: PHP Startup: Unable to load dynamic library '.../event.so' - ..../event.so: undefined symbol: php_sockets_le_socket in Unknown on line 0`.
Veuillez modifier l'ordre de chargement de event.so et socket.so, c'est-à-dire ajouter `extension=socket.so` avant `extension=event.so` dans le fichier php.ini pour charger l'extension socket en premier.
