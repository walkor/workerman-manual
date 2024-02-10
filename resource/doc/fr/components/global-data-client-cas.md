# cas
**``` (Exige une version de Workerman >=3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
Remplacement atomique, remplace ```$old_value``` par ```$new_value```.
Le remplacement de la valeur ne peut avoir lieu que si la valeur correspondante à la clé n'a pas été modifiée par d'autres clients depuis la dernière récupération de la valeur par le client actuel.

## Paramètres

 ``` $key ```

Clé. (Par exemple, si ```$global->abc```, alors ```abc``` est la clé)

 ``` $old_value ```

Ancienne valeur

 ``` $new_value ```

Nouvelle valeur

## Valeur de retour
Retourne true en cas de remplacement réussi, sinon retourne false.

## Remarques :

Lorsque plusieurs processus manipulent simultanément la même variable partagée, il peut être nécessaire de tenir compte des problèmes de concurrence.

Par exemple, les processus A et B ajoutent simultanément un membre à une liste d'utilisateurs.
Les variables de liste d'utilisateurs pour les processus A et B sont actuellement définies comme ```$global->user_list = array(1,2,3)```.
Le processus A modifie la variable ```$global->user_list``` en ajoutant un utilisateur 4.
Le processus B modifie la variable ```$global->user_list``` en ajoutant un utilisateur 5.
Le processus A définit la variable ```$global->user_list = array(1,2,3,4)``` avec succès.
Le processus B définit la variable ```$global->user_list = array(1,2,3,5)``` avec succès.
À ce stade, la variable définie par le processus B écrase la variable définie par le processus A, ce qui entraîne une perte de données.

Ceci est dû au fait que la lecture et la définition ne sont pas des opérations atomiques, ce qui entraîne des problèmes de concurrence.
Pour résoudre ce problème de concurrence, l'interface de remplacement atomique cas peut être utilisée.
Avant de modifier une valeur, l'interface cas vérifie si cette valeur a été modifiée par d'autres processus en se basant sur ```$old_value```. Si elle a été modifiée, le remplacement n'est pas effectué et la méthode retourne false. Sinon, le remplacement est effectué et la méthode retourne true.
Voir l'exemple ci-dessous.

 **Note :** 
Certaines données partagées peuvent être modifiées simultanément sans problème, par exemple le montant d'enchère actuel dans un système d'enchères, ou le stock actuel d'un produit.

## Exemple

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Initialiser la liste
$global->user_list = array(1,2,3);

// Ajouter une valeur atomique à user_list
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
