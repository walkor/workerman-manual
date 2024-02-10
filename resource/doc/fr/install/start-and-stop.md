# Démarrage et arrêt

Veuillez noter que les commandes de démarrage et d'arrêt de Workerman s'effectuent toutes dans l'invite de commande.

Pour démarrer Workerman, il est nécessaire tout d'abord d'avoir un fichier d'entrée de démarrage qui définit le port et le protocole sur lequel le service écoute. Vous pouvez vous référer à la section [Exemple de développement simple](../getting-started/simple-example.md) pour plus de détails.

Prenons l'exemple de [workerman-chat](https://www.workerman.net/workerman-chat), dont le fichier d'entrée de démarrage est start.php.

### Démarrage

Démarrer en mode debug (débogage) :

```php start.php start```

Démarrer en mode daemon (démon) :

```php start.php start -d```

### Arrêt

```php start.php stop```

### Redémarrage

```php start.php restart```

### Redémarrage en douceur

```php start.php reload```

### Vérifier l'état

```php start.php status```

### Vérifier l'état de la connexion (nécessite une version de Workerman >=3.5.0)

```php start.php connections```



## Différences entre les modes debug et daemon

1. En mode debug, les fonctions d'affichage telles que echo, var_dump, print, etc. dans le code s'affichent directement dans le terminal.

2. En mode daemon, les fonctions d'affichage telles que echo, var_dump, print, etc. sont redirigées par défaut vers le fichier /dev/null, mais vous pouvez définir le chemin du fichier en utilisant ```Worker::$stdoutFile = '/your/path/file';```.

3. En mode debug, Workerman se ferme et quitte lorsque le terminal est fermé.

4. En mode daemon, Workerman continue de s'exécuter en arrière-plan normalement lorsque le terminal est fermé.

## Qu'est-ce qu'un redémarrage en douceur ?

Voir [Principe du redémarrage en douceur](../faq/reload-principle.md)
