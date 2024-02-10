# incrément
**``` (Require Workerman version>=3.3.0) ```**
```php
bool \GlobalData\Client::increment(string $key[, int $step = 1])
```
Incrémente de manière atomique. Augmente la valeur d'un élément numérique par la taille spécifiée dans le paramètre step. Si la valeur de l'élément n'est pas de type numérique, elle est traitée comme 0 avant l'incrémentation. Si l'élément n'existe pas, false est retourné.

## Paramètres

 ``` $key ```

Clé de l'élément. (Par exemple, pour ```$global->abc```, ```abc``` est la clé)

 ``` $value ```

Taille à incrémenter à la valeur de l'élément.

## Valeur de retour
Retourne true en cas de succès, sinon retourne false.

## Exemple

```php
$global = new GlobalData\Client('127.0.0.1:2207');

$global->some_key = 0;

// Incrément non atomique
$global->some_key++;

echo $global->some_key."\n";

// Incrément atomique
$global->increment('some_key');

echo $global->some_key."\n";
```
