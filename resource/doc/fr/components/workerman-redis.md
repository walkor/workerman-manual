# workerman-redis

## Introduction

workeman/redis est un composant redis asynchrone basé sur Workerman.

> **Remarque**
> L'objectif principal de ce projet est d'implémenter la souscription asynchrone (subscribe、pSubscribe) à Redis.
> Comme Redis est assez rapide, sauf en cas de besoin de souscriptions asynchrones pSubscribe et subscribe, il n'est pas nécessaire d'utiliser ce client asynchrone. L'utilisation de l'extension Redis offrira de meilleures performances.

## Installation：
```bash
composer require workerman/redis
```

## Utilisation de rappels

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');
    $redis->get('key', function ($result) use ($connection) {
        $connection->send($result);
    });  
};

Worker::runAll();
```

## Utilisation des coroutines

> **Remarque**
> L'utilisation des coroutines nécessite Workerman >= 5.0, workerman/redis >= 2.0.0 et l'installation de composer require revolt/event-loop ^1.0.0.

```php
use Workerman\Worker;
use Workerman\Redis\Client;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:6161');

$worker->onWorkerStart = function() {
    global $redis;
    $redis = new Client('redis://127.0.0.1:6379');
};

$worker->onMessage = function(TcpConnection $connection, $data) {
    global $redis;
    $redis->set('key', 'hello world');
    $result = $redis->get('key');
    $connection->send($result);
};

Worker::runAll();
```

Lorsqu'aucune fonction de rappel n'est définie, le client traitera de manière synchrone les résultats des requêtes asynchrones, sans bloquer le processus actuel, ce qui permet de gérer les requêtes en parallèle.

> **Remarque**
> L'utilisation des coroutines n'est pas prise en charge pour psubscribe subscribe.

# Documentation
**Note**

**Dans une utilisation avec rappels, les fonctions de rappel ont généralement 2 paramètres ($result, $redis), où `$result` représente le résultat et `$redis` représente l'instance de redis. Par exemple:**
```php
use Workerman\Redis\Client;
$redis = new Client('redis://127.0.0.1:6379');
// Définir une fonction de rappel pour vérifier le résultat de l'appel de set
$redis->set('key', 'value', function ($result, $redis) {
    var_dump($result); // true
});
// Les fonctions de rappel sont facultatives, elles sont omises ici
$redis->set('key1', 'value1');
// Les fonctions de rappel peuvent être imbriquées
$redis->get('key', function ($result, $redis){
    $redis->set('key2', 'value2', function ($result) {
        var_dump($result);
    });
});
```

## **Connexion**
```php
use Workerman\Redis\Client;
// Omettre le rappel
$redis = new Client('redis://127.0.0.1:6379');
// Avec le rappel
$redis = new Client('redis://127.0.0.1:6379', [
    'connect_timeout' => 10 // définir un délai de connexion de 10 secondes, 5 secondes par défaut si non défini
], function ($success, $redis) {
    // Rappel du résultat de la connexion
    if (!$success) echo $redis->error();
});
```

## **auth**
```php
// Authentification avec mot de passe
$redis->auth('password', function ($result) {
    
});
// Authentification avec nom d'utilisateur et mot de passe
$redis->auth('username', 'password', function ($result) {

});
```

## **pSubscribe**

Souscrit à un ou plusieurs canaux correspondant au motif donné.

Chaque motif utilise \* comme caractère générique. Par exemple, it\* correspond à tous les canaux commençant par it (it.news, it.blog, it.tweets, etc.). news.\* correspond à tous les canaux commençant par news. (news.it, news.global.today, etc.), et ainsi de suite.

Remarque : La fonction de rappel de pSubscribe comporte 4 paramètres ($pattern, $channel, $message, $redis).

Lorsque l'instance $redis appelle l'interface pSubscribe ou subscribe, toute autre méthode appelée par cette instance sera ignorée.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->psubscribe(['news*', 'blog*'], function ($pattern, $channel, $message) {
    echo "$pattern, $channel, $message"; // news*, news.add, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news.add', 'news content');
});
```

## **subscribe**

Utilisé pour souscrire aux messages des canaux spécifiés.

Remarque : La fonction de rappel de subscribe comporte 3 paramètres ($channel, $message, $redis).

Lorsque l'instance $redis appelle l'interface pSubscribe ou subscribe, toute autre méthode appelée par cette instance sera ignorée.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->subscribe(['news', 'blog'], function ($channel, $message) {
    echo "$channel, $message"; // news, news content
});

Timer::add(5, function () use ($redis2){
    $redis2->publish('news', 'news content');
});
```

## **publish**

Utilisé pour envoyer des messages à un canal spécifique.

Renvoie le nombre d'abonnés ayant reçu le message.

```php
$redis2->publish('news', 'news content');
```

## **select**
```php
// Omettre le rappel
$redis->select(2);
$redis->select('test', function ($result, $redis) {
    // Le paramètre select doit être un nombre, donc $result ici sera faux
    var_dump($result, $redis->error());
});
```

## **get**

Utilisé pour obtenir la valeur de la clé spécifiée. Si la clé n'existe pas, NULL est renvoyé. Si la valeur stockée dans la clé n'est pas de type chaîne, false est renvoyé.

```php
$redis->get('key', function($result) {
     // Renvoie NULL si la clé n'existe pas, sinon false en cas d'erreur
    var_dump($result);
});
```

## **set**

Utilisé pour définir la valeur de la clé spécifiée. Si la clé existe déjà, SET remplacera la vieille valeur, quel que soit le type.

```php
$redis->set('key', 'value');
$redis->set('key', 'value', function($result){});
// Le troisième paramètre peut être un délai d'expiration, expiration après 10 seconds
$redis->set('key','value', 10);
$redis->set('key','value', 10, function($result){});
```

## **setEx, pSetEx**

Définit la valeur d'une clé spécifiée avec son délai d'expiration. Si la clé existe déjà, la commande SETEX remplacera la vieille valeur.

```php
// Remarque : le deuxième paramètre spécifie le délai d'expiration en secondes
$redis->setEx('key', 3600, 'value'); 
// pSetEx spécifie le délai en millisecondes
$redis->pSetEx('key', 3600, 'value'); 
```

## **del**

Utilisé pour supprimer des clés existantes, renvoyant le nombre de clés supprimées (les clés inexistantes ne sont pas comptées).

```php
// Supprime une clé
$redis->del('key');
// Supprime plusieurs clés
$redis->del(['key', 'key1', 'key2']);
```

## **setNx**

(**SET**if**N**ot e**X**ists) : sert à définir une valeur de clé spécifiée si elle n'existe pas.

```php
$redis->del('key');
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 1
});
$redis->setNx('key', 'value', function($result){
    var_dump($result); // 0
});
```

## **exists**

Utilisé pour vérifier si une clé spécifiée existe. Renvoie le nombre de clés existantes.

```php
$redis->set('key', 'value');
$redis->exists('key', function ($result) {
    var_dump($result); // 1
}); 
$redis->exists('NonExistingKey', function ($result) {
    var_dump($result); // 0
}); 

$redis->mset(['foo' => 'foo', 'bar' => 'bar', 'baz' => 'baz']);
$redis->exists(['foo', 'bar', 'baz'], function ($result) {
    var_dump($result); // 3
}); 
```

## **incr, incrBy**

Incrémente d'une unité/du montant spécifié la valeur stockée dans une clé. Si la clé n'existe pas, sa valeur est d'abord initialisée à 0 avant l'opération incr/incrBy.

Si la valeur est de type erroné, ou si la valeur de type chaîne ne peut pas être représentée en tant que nombre, false est renvoyé.

Le résultat renvoyé est la nouvelle valeur après l'incrémentation.

```php
$redis->incr('key1', function ($result) {
    var_dump($result);
}); 
$redis->incrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **incrByFloat**

Incrémente la valeur stockée dans une clé d'une valeur flottante spécifiée.

Si la clé n'existe pas, elle est d'abord initialisée à 0 avant l'opération d'incrémentation par flottant.

Si la valeur est de type erroné, ou si la valeur de type chaîne ne peut pas être représentée en tant que nombre, false est renvoyé.

Le résultat renvoyé est la nouvelle valeur après l'incrémentation.

```php
$redis->incrByFloat('key1', 1.5, function ($result) {
    var_dump($result);
}); 
```

## **decr, decrBy**

Décrémente d'une unité/d'une valeur spécifiée la valeur stockée dans une clé. Si la clé n'existe pas, sa valeur est d'abord initialisée à 0 avant l'opération decr/decrBy.

Si la valeur est de type erroné, ou si la valeur de type chaîne ne peut pas être représentée en tant que nombre, false est renvoyé.

Le résultat renvoyé est la nouvelle valeur après la décrémentation.

```php
$redis->decr('key1', function ($result) {
    var_dump($result);
}); 
$redis->decrBy('key1', 10, function ($result) {
    var_dump($result);
}); 
```

## **mGet**

Renvoie toutes (une ou plusieurs) les valeurs des clés spécifiées. Si certaines clés n'existent pas, elles sont renvoyées comme NULL.

```php
$redis->set('key1', 'value1');
$redis->set('key2', 'value2');
$redis->set('key3', 'value3');
$redis->mGet(['key0', 'key1', 'key5'], function ($result) {
    var_dump($result); // [null, 'value1', null];
}); 
```
## **getSet**

Utilisé pour définir la valeur d'une clé spécifique et renvoyer l'ancienne valeur de la clé.

```php
$redis->set('x', '42');
$redis->getSet('x', 'lol', function ($result) {
    var_dump($result); // '42'
});
$redis->get('x', function ($result) {
    var_dump($result); // 'lol'
});
```

## **randomKey**

Renvoie de manière aléatoire une clé de la base de données actuelle.
```php
$redis->randomKey(function($key) use ($redis) {
    $redis->get($key, function ($result) {
        var_dump($result); 
    });
});
```

## **move**

Déplace la clé de la base de données actuelle vers la base de données db spécifiée.
```php
$redis->select(0);	// basculer vers la base de données 0
$redis->set('x', '42');	// écrire 42 dans x
$redis->move('x', 1, function ($result) { 	// déplacer vers la base de données 1
    var_dump($result); // 1
});  
$redis->select(1);	// basculer vers la base de données 1
$redis->get('x', function ($result) {
    var_dump($result); // '42'
});
```

## **rename**

Modifie le nom de la clé ; retourne false si la clé n'existe pas.
```php
$redis->set('x', '42');
$redis->rename('x', 'y', function ($result) {
    var_dump($result); // true
});
```

## **renameNx**

Modifie le nom de la clé lorsque le nouveau nom de clé n'existe pas.
```php
$redis->del('y');
$redis->set('x', '42');
$redis->renameNx('x', 'y', function ($result) {
    var_dump($result); // 1
});
```

## **expire**

Définit le temps d'expiration de la clé en secondes ; après expiration, la clé ne sera plus disponible. Renvoie 1 en cas de succès, 0 si la clé n'existe pas, et false en cas d'erreur.
```php
$redis->set('x', '42');
$redis->expire('x', 3);
```

## **keys**

Utilisé pour rechercher toutes les clés correspondant au modèle pattern spécifié.
```php
$redis->keys('*', function ($keys) {
    var_dump($keys); 
});
$redis->keys('user*', function ($keys) {
    var_dump($keys); 
});
```

## **type**

Retourne le type de la valeur stockée de la clé. Le résultat retourné est l'une des chaînes suivantes : string, set, list, zset, hash, ou none si la clé n'existe pas.
```php
$redis->type('key', function ($result) {
    var_dump($result); // string set list zset hash none
});
```

## **append**

Si la clé existe déjà et est une chaîne, la commande APPEND ajoutera la valeur à la fin de la valeur existante de la clé, et renverra la longueur de la chaîne. Si la clé n'existe pas, APPEND définira simplement la clé spécifiée sur la valeur fournie, et renverra la longueur de la chaîne. Si la clé existe mais n'est pas une chaîne, false sera renvoyé.
```php
$redis->set('key', 'value1');
$redis->append('key', 'value2', function ($result) {
    var_dump($result); // 12
}); 
$redis->get('key', function ($result) {
    var_dump($result); // 'value1value2'
});
```

## **getRange**

Obtient la sous-chaîne de la chaîne stockée dans la clé spécifiée. La plage de découpe est déterminée par les deux décalages start et end (inclus). Si la clé n'existe pas, une chaîne vide est retournée. Si la clé n'est pas de type chaîne, false est retourné.
```php
$redis->set('key', 'string value');
$redis->getRange('key', 0, 5, function ($result) {
    var_dump($result); // 'string'
});
$redis->getRange('key', -5, -1 , function ($result) {
    var_dump($result); // 'value'
});
```

## **setRange**

Remplace la valeur de la chaîne stockée dans la clé spécifiée par la chaîne spécifiée, à partir de l'offset. Si la clé n'existe pas, elle sera définie avec la chaîne spécifiée. Si la clé n'est pas de type chaîne, false est retourné.
Le résultat retourné est la longueur de la chaîne modifiée.
```php
$redis->set('key', 'Hello world');
$redis->setRange('key', 6, "redis", function ($result) {
    var_dump($result); // 11
});
$redis->get('key', function ($result) {
    var_dump($result); // 'Hello redis'
});
```

## **strLen**

Obtient la longueur de la valeur de la chaîne stockée dans la clé spécifiée. Lorsque la clé stocke une valeur qui n'est pas une chaîne, false est retourné.
```php
$redis->set('key', 'value');
$redis->strlen('key', function ($result) {
    var_dump($result); // 5
});
```

## **getBit**

Pour la valeur de la chaîne stockée dans la clé spécifiée, obtient le bit spécifié par l'offset.```php
$redis->set('key', "\x7f"); // ici, c'est 0111 1111
$redis->getBit('key', 0, function ($result) {
    var_dump($result); // 0
});
```

## **setBit**

Pour la valeur de la chaîne stockée dans la clé spécifiée, définit ou efface le bit spécifié par l'offset. Renvoie 0 ou 1, qui est la valeur avant la modification.
```php
$redis->set('key', "*");	// ord("*") = 42 = 0x2a = "0010 1010"
$redis->setBit('key', 5, 1, function ($result) {
    var_dump($result); // 0
});
```

## **bitOp**

Exécute une opération bit à bit entre plusieurs clés (contenant des valeurs de chaîne) et stocke le résultat dans la clé de destination.
La commande BITOP supporte quatre opérations bit à bit : **ET**, **OU**, **XOR** et **NOT**.
Le résultat retourné est la taille de la chaîne stockée dans la clé de destination, c'est-à-dire la taille de la chaîne d'entrée la plus longue.
```php
$redis->set('key1', "abc");
$redis->bitOp( 'AND', 'dst', 'key1', 'key2', function ($result) {
    var_dump($result); // 3
});
```

## **bitCount**

Calcule le nombre de bits définis (population count) dans la chaîne.
Par défaut, tous les octets de la chaîne sont examinés. Vous ne pouvez spécifier qu'une plage d'octets supplémentaire avec les arguments optionnels start et end.
Tout comme avec la commande GETRANGE, les positions de début et de fin peuvent être négatives pour indexer les octets à partir de la fin de la chaîne, où -1 est le dernier octet, -2 est l'avant-dernier octet, etc.  
Le résultat retourné est le nombre de bits définis à 1 dans la chaîne. Les clés inexistantes sont traitées comme des chaînes vides et renverront donc zéro.
```php
$redis->set('key', 'hello');
$redis->bitCount( 'key', 0, 0, function ($result) {
    var_dump($result); // 3
});
$redis->bitCount( 'key', function ($result) {
    var_dump($result); //21
});
```

## **sort**

La commande sort trie les éléments d'une liste, d'un ensemble ou d'un ensemble ordonné.
Prototype : `sort($key, $options, $callback);`
Où les options sont les paires clé-valeur optionnelles suivantes :
~~~
$options = [
     'by' => 'some_pattern_*',
    'limit' => [0, 1],
    'get' => 'some_other_pattern_*', // ou un tableau de modèles
    'sort' => 'asc', // ou 'desc'
    'alpha' => true,
    'store' => 'external-key'
];
~~~
```php
$redis->del('s');
$redis->sAdd('s', 5);
$redis->sAdd('s', 4);
$redis->sAdd('s', 2);
$redis->sAdd('s', 1);
$redis->sAdd('s', 3);
$redis->sort('s', [], function ($result) {
    var_dump($result); // 1,2,3,4,5
});
$redis->sort('s', ['sort' => 'desc'], function ($result) {
    var_dump($result); // 5,4,3,2,1
});
$redis->sort('s', ['sort' => 'desc', 'store' => 'out'], function ($result) {
    var_dump($result); // (int)5
});
```


## **ttl, pttl**

Renvoie le temps d'expiration restant de la clé en secondes/millisecondes.
Si la clé n'a pas de TTL, -1 est renvoyé. Si la clé n'existe pas, -2 est renvoyé.
```php
$redis->set('key', 'value', 10);
// en secondes
$redis->ttl('key', function ($result) {
    var_dump($result); // 10
});
// en millisecondes
$redis->pttl('key', function ($result) {
    var_dump($result); // 9999
});
// la clé n'existe pas
$redis->pttl('key-not-exists', function ($result) {
    var_dump($result); // -2
});
```

## **persist**

Supprime le temps d'expiration de la clé spécifiée, la rendant ainsi persistante (non expirante).
Si success, renvoie 1. Si la clé n'existe pas ou n'a pas de temps d'expiration, renvoie 0. Si une erreur se produit, renvoie false.
```php
$redis->persist('key');
```


## **mSet, mSetNx**

Définit plusieurs paires clé-valeur dans une commande atomique. Dans le cas de mSetNx, retourne 1 uniquement si toutes les clés sont définies.
Renvoie 1 en cas de succès, 0 en cas d'échec, et false en cas d'erreur.
```php
$redis->mSet(['key0' => 'value0', 'key1' => 'value1']);
```


## **hSet**

Affecte une valeur à un champ spécifique dans un hachage. 
Si le champ est un nouveau champ dans le hachage et que la valeur est définie avec succès, renvoie 1. Si le champ de hachage existe déjà et que l'ancienne valeur a été remplacée par la nouvelle valeur, renvoie 0.
```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // hello
});
$redis->hSet('h', 'key1', 'plop', function ($r) {
    var_dump($r); // 0
});
$redis->hGet('h', 'key1', function ($r) {
    var_dump($r); // plop
});
```


## **hSetNx**

Affecte une valeur à un champ dans un hachage qui n'existe pas encore. 
Si le hachage n'existe pas, un nouveau hachage est créé et l'opération HSET est effectuée.
Si le champ existe déjà dans le hachage, l'opération est invalide.
Si la clé n'existe pas, un nouveau hachage est créé et l'opération HSETNX est effectuée.
```php
$redis->del('h');
$redis->hSetNx('h', 'key1', 'hello', function ($r) {
    var_dump($r); // 1
});
$redis->hSetNx('h', 'key1', 'world', function ($r) {
    var_dump($r); // 0
});
```


## **hGet**

Renvoie la valeur d'un champ spécifique dans un hachage. 
Si le champ ou la clé spécifiée n'existe pas, renvoie null.
```php
$redis->hGet('h', 'key1', function ($result) {
    var_dump($result);
});
```
## **hLen**

Utilisé pour obtenir le nombre de champs dans une table de hachage.

Lorsque la clé n'existe pas, retourne 0.

```php
$redis->del('h');
$redis->hSet('h', 'key1', 'hello');
$redis->hSet('h', 'key2', 'plop');
$redis->hLen('h', function ($result) {
    var_dump($result); // 2
});
```

## **hDel**

La commande est utilisée pour supprimer un ou plusieurs champs spécifiés de la table de hachage key, les champs inexistants seront ignorés.

Renvoie le nombre de champs supprimés avec succès, à l'exclusion des champs ignorés. Si la clé n'est pas une table de hachage, renvoie faux.

```php
$redis->hDel('h', 'key1');
```

## **hKeys**

Récupère tous les membres du hachage dans un tableau.

Si la clé n'existe pas, alors renvoie un tableau vide. Si la clé n'est pas un hachage, alors renvoie faux.

```php
$redis->hKeys('key', function ($result) {
    var_dump($result);
});
```

## **hVals**

Retourne sous forme de tableau toutes les valeurs des champs d'un hachage.

Si la clé n'existe pas, renvoie un tableau vide. Si la clé n'est pas un hachage, alors renvoie faux.

```php
$redis->hVals('key', function ($result) {
    var_dump($result);
});
```

## **hGetAll**

Retourne tous les champs et leurs valeurs d'un hachage sous forme d'un tableau associatif.

Si la clé n'existe pas, retourne un tableau vide. Si la clé n'est pas un hachage, alors renvoie faux.

```php
$redis->del('h');
$redis->hSet('h', 'a', 'x');
$redis->hSet('h', 'b', 'y');
$redis->hSet('h', 'c', 'z');
$redis->hSet('h', 'd', 't');
$redis->hGetAll('h', function ($result) {
    var_export($result); 
});
```
Renvoie
```php
array (
    'a' => 'x',
    'b' => 'y',
    'c' => 'z',
    'd' => 't',
)
```

## **hExists**

Vérifie si un champ spécifié existe dans le hachage. Renvoie 1 si le champ existe, 0 si le champ ou la clé n'existe pas, et faux en cas d'erreur.

```php
$redis->hExists('h', 'a', function ($result) {
    var_dump($result); //
});
```

## **hIncrBy**

Utilisé pour ajouter une valeur incrémentale spécifiée aux champs d'un hachage.

L'incrément peut également être négatif, équivalent à une opération de soustraction sur le champ spécifié.

Si la clé du hachage n'existe pas, un nouveau hachage est créé et la commande HINCRBY est exécutée.

Si le champ spécifié n'existe pas, la valeur du champ est initialisée à 0 avant l'exécution de la commande.

Renvoie faux si la commande HINCRBY est exécutée sur un champ contenant une valeur de type chaîne.

La valeur de cette opération est limitée à 64 bits en représentation numérique signée.

```php
$redis->del('h');
$redis->hIncrBy('h', 'x', 2,  function ($result) {
    var_dump($result);
});
```

## **hIncrByFloat**

Similaire à hIncrBy, mais l'incrément est un nombre à virgule flottante.

## **hMSet**

Définit simultanément plusieurs paires champ-valeur dans le hachage.

Cette commande écrase les champs existants dans le hachage.

Si le hachage n'existe pas, un hachage vide est créé et l'opération HMSET est exécutée.

```php
$redis->del('h');
$redis->hMSet('h', ['name' => 'Joe', 'sex' => 1])
```

## **hMGet**

Renvoie les valeurs des champs spécifiés dans un hachage sous forme de tableau associatif.

Si le champ spécifié n'existe pas dans le hachage, la valeur correspondante est null. Si la clé n'est pas un hachage, renvoie faux.

```php
$redis->del('h');
$redis->hSet('h', 'field1', 'value1');
$redis->hSet('h', 'field2', 'value2');
$redis->hMGet('h', ['field1', 'field2', 'field3'], function ($r) {
    var_export($r);
});
```
Sortie
```php
array (
 'field1' => 'value1',
 'field2' => 'value2',
 'field3' => null
)
```
## **blPop, brPop**

Retire et récupère le premier élément/dernier élément de la liste. Si la liste est vide, la commande bloquera jusqu'à ce qu'un élément puisse être récupéré ou que le délai d'attente soit dépassé.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');

$redis->blPop(['key1', 'key2'], 10, function ($r) {
    var_export($r); // array ( 0 => 'key1',1 => 'a')
});

Timer::add(1, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
});
```

## **bRPopLPush**

Récupère le dernier élément de la liste et l'insère au début d'une autre liste ; Si la liste est vide, la commande bloquera jusqu'à ce qu'un élément puisse être transféré ou que le délai d'attente soit dépassé. Renvoie null en cas de dépassement de délai.

```php
$redis = new Client('redis://127.0.0.1:6379');
$redis2 = new Client('redis://127.0.0.1:6379');
$redis->del(['key1', 'key2']);
$redis->bRPopLPush('key1', 'key2', 2, function ($r) {
    var_export($r);
});
Timer::add(2, function () use ($redis2) {
    $redis2->lpush('key1', 'a');
    $redis2->lRange('key2', 0, -1, function ($r) {
        var_dump($r);
    });
}, null, false);
```

## **lIndex**

Récupère un élément de la liste par index. Vous pouvez également utiliser des indices négatifs, où -1 représente le dernier élément de la liste, -2 représente l'avant-dernier élément, et ainsi de suite.

Si l'index spécifié est en dehors de la plage de la liste, renvoie null. Si la clé n'est pas de type liste, renvoie faux.

```php
$redis->del('key1']);
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lindex('key1', 0, function ($r) {
    var_dump($r); // A
});
```

## **lInsert**

Insère un élément avant ou après un autre élément de la liste. Aucune opération n'est effectuée si l'élément spécifié n'existe pas dans la liste.

Lorsque la liste n'existe pas, elle est considérée comme une liste vide et aucune opération n'est effectuée.

Renvoie faux si la clé n'est pas de type liste.

```php
$redis->del('key1');
$redis->lInsert('key1', 'after', 'A', 'X', function ($r) {
    var_dump($r); // 0
});
$redis->lPush('key1', 'A');
$redis->lPush('key1', 'B');
$redis->lPush('key1', 'C');
$redis->lInsert('key1', 'before', 'C', 'X', function ($r) {
    var_dump($r); // 4
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'X', 'C']
});
```

## **lPop**

Supprime et renvoie le premier élément de la liste.

Lorsque la clé de la liste n'existe pas, renvoie null.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->lPop('key1', function ($r) {
    var_dump($r); // A
});
```

## **lPush**

Insère une ou plusieurs valeurs au début de la liste. Si la clé n'existe pas, une liste vide est créée et la commande LPUSH est exécutée. Renvoie faux si la clé existe mais n'est pas de type liste.

**Remarque :** Avant la version 2.4 de Redis, la commande LPUSH ne prenait en charge qu'une valeur unique.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPush('key1', ['B','C']);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lPushx**

Insère une valeur au début d'une liste existante, cette opération est sans effet si la liste n'existe pas, et renvoie 0. Renvoie faux si la clé n'est pas de type liste.

La valeur retournée par la commande lPushx est la longueur de la liste après l'exécution de lPushx.

```php
$redis->del('key1');
$redis->lPush('key1', 'A');
$redis->lPushx('key1', ['B','C'], function ($r) {
    var_dump($r); // 3
});
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRange**

Renvoie une liste des éléments dans la plage spécifiée de la liste. La plage est définie par les décalages START et END. 0 représente le premier élément de la liste, 1 le deuxième élément, et ainsi de suite. Vous pouvez également utiliser des indices négatifs pour les éléments à partir de la fin de la liste.

Renvoie un tableau contenant les éléments spécifiés dans la plage donnée. Renvoie faux si la clé n'est pas de type liste.

```php
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['C', 'B', 'A']
});
```

## **lRem**

En fonction de la valeur COUNT, supprime les éléments dans la liste qui correspondent à la valeur spécifiée.

La valeur de COUNT peut être :

* count > 0 : Commence par le début de la liste et supprime COUNT éléments ayant la valeur VALUE.
* count < 0 : Commence par la fin de la liste et supprime COUNT éléments ayant la valeur VALUE.
* count = 0 : Supprime tous les éléments ayant la valeur VALUE de la liste.

Renvoie le nombre d'éléments supprimés. Renvoie 0 si la liste n'existe pas. Renvoie faux si la clé n'est pas une liste.

```php
$redis->lRem('key1', 2, 'A', function ($r) {
    var_dump($r); 
});
```

## **lSet**

Définit la valeur d'un élément dans une liste en fonction de l'indice.

Renvoie vrai en cas de succès. Renvoie faux si l'indice est en dehors de la plage de la liste ou si la liste est vide.

```php
$redis->lSet('key1', 0, 'X');
```
## **lTrim**

La méthode lTrim est utilisée pour tailler (trim) une liste, c'est-à-dire de ne conserver que les éléments dans une plage spécifiée et de supprimer tous les éléments en dehors de cette plage.

L'indice 0 représente le premier élément de la liste, 1 représente le deuxième élément de la liste, et ainsi de suite. Vous pouvez également utiliser des indices négatifs, -1 représentant le dernier élément de la liste, -2 représentant l'avant-dernier élément de la liste, et ainsi de suite.

En cas de succès, elle retourne true, et en cas d'échec, elle retourne false.

```php
$redis->del('key1');

$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B', 'C']
});
$redis->lTrim('key1', 0, 1);
$redis->lRange('key1', 0, -1, function ($r) {
    var_dump($r); // ['A', 'B']
});
```

## **rPop**

Utilisé pour retirer le dernier élément d'une liste et renvoyer la valeur supprimée.

En cas d'absence de la liste, il retourne null.

```php
$redis->rPop('key1', function ($r) {
    var_dump($r);
});
```

## **rPopLPush**

Utilisé pour retirer le dernier élément d'une liste et l'ajouter à une autre liste, puis renvoyer la valeur.

```php
$redis->del('x', 'y');
$redis->lPush('x', 'abc');
$redis->lPush('x', 'def');
$redis->lPush('y', '123');
$redis->lPush('y', '456');
$redis->rPopLPush('x', 'y', function ($r) {
    var_dump($r); // abc
});
$redis->lRange('x', 0, -1, function ($r) {
    var_dump($r); // ['def']
});
$redis->lRange('y', 0, -1, function ($r) {
    var_dump($r); // ['abc', '456', '123']
});
```

## **rPush**

Ajoute une ou plusieurs valeurs à la fin d'une liste et retourne la longueur de la liste après l'ajout.

Si la liste n'existe pas, une liste vide est créée et l'opération RPUSH est effectuée. Si la liste existe mais n'est pas de type liste, elle retourne false.

**Remarque :** Avant la version Redis 2.4, la commande RPUSH n'acceptait qu'une seule valeur.

```php
$redis->del('key1');
$redis->rPush('key1', 'A', function ($r) {
    var_dump($r); // 1
});
```

## **rPushX**

Ajoute une valeur à la fin d'une liste existante et retourne la longueur de la liste. Si la liste n'existe pas, l'opération n'est pas effectuée et elle retourne 0. Si la liste existe mais n'est pas de type liste, elle retourne false.

```php
$redis->del('key1');
$redis->rPushX('key1', 'A', function ($r) {
    var_dump($r); // 0
});
```

## **lLen**

Renvoie la longueur d'une liste. Si la clé de la liste n'existe pas, elle est interprétée comme étant une liste vide et renvoie 0. Si la clé n'est pas de type liste, elle retourne false.

```php
$redis->del('key1');
$redis->rPush('key1', 'A');
$redis->rPush('key1', 'B');
$redis->rPush('key1', 'C');
$redis->lLen('key1', function ($r) {
    var_dump($r); // 3
});
```

## **sAdd**

Ajoute un ou plusieurs membres à un ensemble. Les membres déjà présents dans l'ensemble seront ignorés.

Si la clé de l'ensemble n'existe pas, un ensemble contenant uniquement les éléments ajoutés comme membres est créé.

Si la clé de l'ensemble n'est pas de type ensemble, elle retourne false.

**Remarque :** Avant Redis 2.4, SADD n'acceptait qu'un seul membre.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , ['member2', 'member3'], function ($r) {
    var_dump($r); // 2
});
$redis->sAdd('key1' , 'member2', function ($r) {
    var_dump($r); // 0
});
```

## **sCard**

Renvoie le nombre d'éléments dans un ensemble. Si la clé de l'ensemble n'existe pas, elle retourne 0.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3');
$redis->sCard('key1', function ($r) {
    var_dump($r); // 3
});
$redis->sCard('keyX', function ($r) {
    var_dump($r); // 0
});
```

## **sDiff**

Renvoie la différence entre le premier ensemble et les autres ensembles, c'est-à-dire les éléments exclusivement présents dans le premier ensemble. Les ensembles inexistants seront considérés comme vides.

```php
$redis->del('s0', 's1', 's2');

$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s0', '3');
$redis->sAdd('s0', '4');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sDiff(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // ['2', '4']
});
```

## **sDiffStore**

Stoque la différence entre les ensembles spécifiés dans un ensemble spécifié. Si la clé de l'ensemble de destination existe déjà, elle est écrasée.

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s0', '3');
$redis->sAdd('s0', '4');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sDiffStore('dst', ['s0', 's1', 's2'], function ($r) {
    var_dump($r); // 2
});
$redis->sMembers('dst', function ($r) {
    var_dump($r); // ['2', '4']
});
```

## **sInter**

Renvoie l'intersection de tous les ensembles spécifiés. Les ensembles inexistants sont considérés comme étant vides. Si l'un des ensembles est vide, le résultat est également vide.

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('key1', 'val1');
$redis->sAdd('key1', 'val2');
$redis->sAdd('key1', 'val3');
$redis->sAdd('key1', 'val4');
$redis->sAdd('key2', 'val3');
$redis->sAdd('key2', 'val4');
$redis->sAdd('key3', 'val3');
$redis->sAdd('key3', 'val4');
$redis->sInter(['key1', 'key2', 'key3'], function ($r) {
    var_dump($r); // ['val4', 'val3']
});
```

## **sInterStore**

Stoque l'intersection de plusieurs ensembles dans un ensemble spécifié et renvoie le nombre d'éléments dans l'ensemble résultant. Si l'ensemble spécifié existe déjà, il est écrasé.

```php
$redis->sAdd('key1', 'val1');
$redis->sAdd('key1', 'val2');
$redis->sAdd('key1', 'val3');
$redis->sAdd('key1', 'val4');

$redis->sAdd('key2', 'val3');
$redis->sAdd('key2', 'val4');

$redis->sAdd('key3', 'val3');
$redis->sAdd('key3', 'val4');

$redis->sInterStore('output', 'key1', 'key2', 'key3', function ($r) {
    var_dump($r); // 2
});
$redis->sMembers('output', function ($r) {
    var_dump($r); // ['val4', 'val3']
});
```

## **sIsMember**

Vérifie si un membre est membre de l'ensemble donné.

Si le membre est un membre de l'ensemble, elle retourne 1. Si le membre n'est pas un membre de l'ensemble ou si la clé n'existe pas, elle retourne 0. Si la clé n'est pas de type ensemble, elle retourne false.

```php
$redis->sIsMember('key1', 'member1', function ($r) {
    var_dump($r); 
});
```

## **sMembers**

Retourne tous les membres de l'ensemble spécifié. Si l'ensemble n'existe pas, elle retourne un ensemble vide.

```php
$redis->sMembers('s', function ($r) {
    var_dump($r); 
});
```

## **sMove**

Déplace un membre spécifié d'un ensemble source à un ensemble de destination.

SMOVE est une opération atomique. Si l'ensemble source n'existe pas ou ne contient pas le membre spécifié, la commande SMOVE ne fait rien et retourne simplement 0. Sinon, le membre est retiré de l'ensemble source et ajouté à l'ensemble de destination. Si l'ensemble de destination contient déjà le membre, SMOVE supprime simplement le membre de l'ensemble source.

Si la source ou la destination n'est pas de type ensemble, elle retourne false.

```php
$redis->sMove('key1', 'key2', 'member13');
```

## **sPop**

Retire un ou plusieurs éléments aléatoires de l'ensemble spécifié et renvoie les éléments retirés.

Si l'ensemble n'existe pas ou est vide, elle retourne null.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3');
$redis->sPop('key1', function ($r) {
    var_dump($r); // member3
});
$redis->sAdd('key2', ['member1', 'member2', 'member3']);
$redis->sPop('key2', 3, function ($r) {
    var_dump($r); // ['member1', 'member2', 'member3']
});
```

## **sRandMember**

La commande Srandmember est utilisée pour retourner un élément aléatoire de l'ensemble spécifié.

À partir de Redis version 2.6, la commande Srandmember accepte un paramètre facultatif count :

*   Si le count est positif et inférieur au cardinal de l'ensemble, la commande renvoie un tableau contenant count éléments distincts. Si count est supérieur ou égal au cardinal de l'ensemble, elle renvoie l'ensemble entier.
*   Si le count est négatif, la commande renvoie un tableau contenant des éléments qui peuvent apparaître plusieurs fois, la longueur du tableau est égale à la valeur absolue de count.

Cette opération est similaire à SPOP, mais tandis que SPOP retire l'élément aléatoire de l'ensemble, Srandmember ne modifie pas l'ensemble et se contente de renvoyer l'élément aléatoire.

```php
$redis->del('key1');
$redis->sAdd('key1' , 'member1');
$redis->sAdd('key1' , 'member2');
$redis->sAdd('key1' , 'member3'); 

$redis->sRandMember('key1', function ($r) {
    var_dump($r); // member1
});

$redis->sRandMember('key1', 2, function ($r) {
    var_dump($r); // ['member1', 'member2']
});
$redis->sRandMember('key1', -100, function ($r) {
    var_dump($r); // ['member1', 'member2', 'member3', 'member3', ...]
});
$redis->sRandMember('empty-set', 100, function ($r) {
    var_dump($r); // []
}); 
$redis->sRandMember('not-a-set', 100, function ($r) {
    var_dump($r); // []
});
```
## **sRem**

Supprime un ou plusieurs membres d'un ensemble, les membres qui n'existent pas seront ignorés.

Retourne le nombre d'éléments supprimés avec succès, sans inclure les éléments ignorés.

Retourne false si la clé n'est pas de type ensemble.

Avant la version 2.4 de Redis, SREM ne prenait en charge qu'une seule valeur de membre.

```php
$redis->sRem('key1', ['member2', 'member3'], function ($r) {
    var_dump($r); 
});
```

## **sUnion**

La commande renvoie l'union des ensembles donnés. Les ensembles inexistants sont considérés comme vides.

```php
$redis->sUnion(['s0', 's1', 's2'], function ($r) {
    var_dump($r); // []
});
```

## **sUnionStore**

Stocke l'union des ensembles donnés dans l'ensemble destination spécifié et retourne le nombre d'éléments. Si la destination existe déjà, elle sera écrasée.

```php
$redis->del('s0', 's1', 's2');
$redis->sAdd('s0', '1');
$redis->sAdd('s0', '2');
$redis->sAdd('s1', '3');
$redis->sAdd('s1', '1');
$redis->sAdd('s2', '3');
$redis->sAdd('s2', '4');
$redis->sUnionStore('dst', 's0', 's1', 's2', function ($r) {
    var_dump($r); // 4
});
$redis->sMembers('dst', function ($r) {
    var_dump($r); // ['1', '2', '3', '4']
});
```

****
