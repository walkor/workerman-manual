# Exigences système

## Utilisateurs Windows
Depuis la version 3.5.3, Workerman prend en charge à la fois les systèmes Linux et Windows.

1. PHP >= 5.4 est nécessaire, et l'environnement PHP doit être configuré correctement.

2. La version Windows de Workerman ne dépend d'aucune extension.

3. Pour l'installation, l'utilisation et les limitations, veuillez consulter [ici](https://www.workerman.net/windows).

4. En raison de nombreuses limitations d'utilisation de Workerman sous Windows, il est recommandé d'utiliser un système Linux en environnement de production, et le système Windows est uniquement recommandé pour le développement.

 ``` ====Les instructions ci-dessous ne s'appliquent qu'aux utilisateurs Linux, veuillez ignorer si vous utilisez Windows. ====```

## Utilisateurs Linux (y compris Mac OS)
Les utilisateurs Linux ne peuvent utiliser que la version Linux de Workerman.

1. Installez PHP >= 5.4 et assurez-vous d'avoir installé les extensions pcntl et posix.

2. Il est recommandé d'installer l'extension event, bien que ce ne soit pas obligatoire (notez que l'extension event nécessite PHP >= 5.4).

### Script de vérification de l'environnement Linux
Les utilisateurs Linux peuvent exécuter le script ci-dessous pour vérifier si leur environnement local répond aux exigences de WorkerMan.

 ```curl -Ss https://www.workerman.net/check | php```

Si toutes les vérifications dans le script sont correctes, cela signifie que l'environnement d'exécution de WorkerMan est correct.

(Note : le script de vérification ne vérifie pas l'extension event. Si le nombre de connexions simultanées est supérieur à 1024, il est recommandé d'installer l'extension event. Veuillez consulter la section suivante pour les instructions d'installation.)

## Informations détaillées

### À propos de PHP-CLI

WorkerMan fonctionne en mode ligne de commande PHP (PHP-CLI). PHP-CLI est un programme exécutable indépendant, sans conflit ni dépendance avec PHP-FPM ou le module PHP d'Apache.

### Extensions requises par WorkerMan

1. [Extension pcntl](https://www.php.net/manual/fr/book.pcntl.php)

L'extension pcntl est importante pour le contrôle des processus PHP sous Linux. WorkerMan fait appel à ses fonctionnalités telles que la création de processus, le contrôle des signaux, les minuteries et la surveillance de l'état des processus. Cette extension n'est pas prise en charge sous Windows.

2. [Extension posix](https://www.php.net/manual/fr/book.posix.php)

L'extension posix permet à PHP d'appeler les interfaces fournies par les systèmes basés sur les normes POSIX. WorkerMan l'utilise principalement pour mettre en œuvre des fonctionnalités telles que la démonisation et le contrôle des groupes d'utilisateurs. Cette extension n'est pas prise en charge sous Windows.

3. [Extension Event](https://www.php.net/manual/fr/book.event.php) ou [Extension libevent](https://www.php.net/manual/fr/book.libevent.php)

L'extension event permet à PHP d'utiliser des mécanismes de traitement d'événements avancés tels que Epoll et Kqueue, ce qui améliore considérablement l'utilisation du processeur par WorkerMan lors de connexions simultanées élevées. Elle est très importante pour les applications impliquant des connexions simultanées à long terme. L'extension libevent (ou event) n'est pas obligatoire ; si elle n'est pas installée, WorkerMan utilisera par défaut le mécanisme de traitement d'événements natif de PHP, Select.

## Comment installer les extensions

Veuillez vous référer à la section [Installation des extensions](../appendices/install-extension.md)
