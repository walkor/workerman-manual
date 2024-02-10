# Protocole text
> Workerman définit un protocole de texte appelé "text", dont le format est ```paquet de données + saut de ligne```, c'est-à-dire qu'à la fin de chaque paquet de données, un saut de ligne est ajouté pour indiquer la fin du paquet.

Par exemple, les chaînes buffer1 et buffer2 suivantes sont conformes au protocole texte :

```php
// Texte avec un retour à la ligne
$buffer1 = 'abcdefghijklmn
';
// Dans PHP, "\n" entre guillemets représente un saut de ligne, par exemple "\n"
$buffer2 = '{"type":"say", "content":"hello"}'."\n";

// Établir une connexion socket avec le serveur
$client = stream_socket_client('tcp://127.0.0.1:5678');
// Envoyer les données du buffer1 en utilisant le protocole text
fwrite($client, $buffer1);
// Envoyer les données du buffer2 en utilisant le protocole text
fwrite($client, $buffer2);
```

Le protocole text est très simple et facile à utiliser. Si un développeur a besoin de son propre protocole, par exemple pour transmettre des données à une application mobile ou pour communiquer avec du matériel, il peut envisager d'utiliser le protocole text, car il est très pratique pour le développement et le débogage.

**Débogage du protocole text**

> Le protocole text peut être débogué à l'aide d'un client telnet. Voici un exemple :

Créez un nouveau fichier test.php

```php
require_once __DIR__ . '/Workerman/Autoloader.php';
use Workerman\Worker;

$text_worker = new Worker("text://0.0.0.0:5678");

$text_worker->onMessage =  function($connection, $data)
{
    var_dump($data);
    $connection->send("hello world");
};

Worker::runAll();
```

Exécutez ```php test.php start``` pour afficher ceci :

```bash
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

Ouvrez un nouveau terminal et utilisez telnet pour tester (il est recommandé d'utiliser telnet sur un système Linux) :

Supposons que vous testez sur la machine locale,
Dans le terminal, exécutez telnet 127.0.0.1 5678
Ensuite, saisissez hi + Entrée
Vous recevrez la donnée hello world\n
```bash
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
