# Combien de processus devraient être activés

## Comment configurer le nombre de processus
Le nombre de processus est déterminé par l'attribut ```count``` (le système Windows ne prend pas en charge la configuration du nombre de processus), par exemple le code ci-dessous :
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## Démarrer 4 processus pour fournir des services externes ##
$http_worker->count = 4;

...
```

## Facteurs à considérer pour configurer le nombre de processus
1. Nombre de cœurs de CPU
2. Taille de la mémoire
3. Orienté IO intensif ou orienté CPU intensif de l'activité commerciale

## Principes de configuration du nombre de processus
1. La somme de la mémoire occupée par chaque processus doit être inférieure à la mémoire totale (en général, chaque processus commercial occupe environ 40 Mo de mémoire).
2. Pour une activité commerciale orientée IO intensif, c'est-à-dire impliquant des opérations IO blocantes telles que l'accès à Mysql, Redis, etc., le nombre de processus peut être augmenté, par exemple, configuré comme étant 3 fois le nombre de cœurs de CPU. Si l'activité implique beaucoup d'attentes bloquantes, le nombre de processus peut être augmenté de manière appropriée, par exemple, 8 fois le nombre de cœurs de CPU ou même plus. Il convient de noter que l'IO non blocant appartient à la catégorie des activités CPU intensives, et non à celle des activités IO intensives.
3. Pour une activité commerçiale orientée CPU intensif, c'est-à-dire sans charge IO bloquante, par exemple l'utilisation de lectures asynchrones IO des ressources réseau, le nombre de processus peut être configuré pour correspondre au nombre de cœurs de CPU.

## Valeurs de référence pour la configuration du nombre de processus
Si le code commercial est orienté IO intensif, alors le nombre de processus doit être configuré en fonction du degré d'IO, par exemple entre 3 et 8 fois le nombre de cœurs de CPU.
Si le code commercial est orienté CPU intensif, alors le nombre de processus peut être configuré pour correspondre au nombre de cœurs de CPU.

## Remarque
Les IO de Workerman lui-même sont non bloquants, par exemple les opérations telles que ```Connection->send``` sont non bloquantes, ce qui relève de l'activité CPU intensive. Si l'orientation de l'activité commerciale n'est pas claire, le nombre de processus peut être configuré à environ 3 fois le nombre de cœurs de CPU.
De plus, plus de processus ne signifie pas nécessairement de meilleurs résultats. Si trop de processus sont ouverts, le surcoût de basculement des processus augmentera, ce qui aura un certain impact sur les performances.
