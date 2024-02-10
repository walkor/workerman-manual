# ajouter
**``` (Nécessite Workerman version>=3.3.0) ```**
```php
bool \GlobalData\Client::add(string $key, mixed $value)
```
Ajout atomique. Si la clé existe déjà, elle renverra false.

## Paramètres

 ``` $key ```

Clé. (Par exemple, si ```$global->abc```, alors ```abc``` est la clé)

 ``` $value ```

Valeur à stocker.

## Valeur de retour
Retourne true si réussi, sinon false.

## Exemple

```php
$global = new GlobalData\Client('127.0.0.1:2207');

if($global->add('some_key', 10))
{
    // La valeur de $global->some_key a été ajoutée avec succès
    echo "ajout réussi " , $global->some_key;
}
else
{
    // La clé $global->some_key existe déjà, l'ajout a échoué
    echo "échec de l'ajout " , var_export($global->some_key);
}
```
