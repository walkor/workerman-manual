# Débogage des processus occupés
Parfois, en exécutant la commande ```php start.php status```, nous pouvons voir des processus ayant l'état ```busy```, ce qui signifie que le processus correspondant est en train de traiter une tâche. Normalement, une fois la tâche terminée, le processus revient à l'état ```idle```, ce qui ne pose généralement pas de problème. Cependant, s'il reste constamment dans l'état ```busy``` sans revenir à l'état ```idle```, cela signifie que le processus est soit bloqué dans une tâche, soit pris dans une boucle infinie. Vous pouvez le localiser en utilisant les méthodes suivantes.

## Localisation à l'aide de la commande strace + lsof

**1. Trouver le PID du processus occupé dans la liste des états**
Après avoir exécuté la commande ```php start.php status```, vous verrez quelque chose comme ceci :
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
Dans cette liste, les PID des processus ```busy``` sont ```11725``` et ```11748```.

**2. Traçage du processus avec strace**
Choisissez l'un des processus avec le PID (par exemple ```11725```) et exécutez ```strace -ttp 11725```, pour afficher quelque chose comme :
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
Vous verrez que le processus effectue une boucle continue d'appels système ```poll([{fd=16, events=....```, ce qui indique qu'il attend un événement de lecture pour le descripteur de fichier 16, c'est-à-dire qu'il attend des données de ce descripteur.

Si aucun appel système n'est affiché, gardez le terminal actuel ouvert, ouvrez un nouveau terminal et exécutez ```kill -SIGALRM 11725``` (envoyer un signal d'alarme au processus), puis vérifiez si le terminal strace répond ou s'il reste bloqué à un appel système. Si aucun appel système n'est toujours affiché, il est probable que le programme soit bloqué dans une boucle infinie. Consultez la section ci-dessous intitulée "Autres causes de la prolongation du processus occupé" point 2 pour résoudre ce problème.

S'il reste bloqué dans l'appel système epoll_wait ou select, c'est tout à fait normal, ce qui signifie que le processus est en état ```idle```.

**3. Vérification des descripteurs de fichier du processus avec lsof**
Exécutez ```lsof -nPp 11725``` pour afficher quelque chose comme :
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
Le descripteur de fichier 16 correspond à l'enregistrement 16u (dernière ligne). Vous pouvez voir que le descripteur ```fd=16``` est une connexion TCP, avec une adresse distante ```101.37.136.135:80```, ce qui indique que le processus doit accéder à une ressource HTTP, et la boucle continue ```poll([{fd=16, events=....``` signifie qu'il attend constamment des données de retour du serveur HTTP, expliquant ainsi pourquoi le processus reste dans l'état ```busy```.

**Résolution :**
Une fois que vous avez identifié où le processus est bloqué, il est plus facile de résoudre le problème. Par exemple, dans le cas ci-dessus, après avoir localisé le problème, il est probable que l'application appelle la fonction curl, mais que l'URL correspondante met trop de temps à retourner des données, bloquant ainsi le processus. Dans ce cas, vous devriez contacter le fournisseur de l'URL pour comprendre la raison du retard de retour, tout en ajoutant un délai d'attente lors de l'appel de curl, par exemple un délai de 2 secondes. Cela évitera le blocage prolongé du processus (et le maintiendra dans l'état ```busy``` pendant environ 2 secondes).

## Autres causes de la prolongation du processus occupé
En plus du blocage du processus ou des boucles infinies dans le code, il existe d'autres raisons pour lesquelles un processus reste dans l'état ```busy```.

**1. Erreurs fatales dans le code de l'application provoquant la sortie répétée du processus**
**Phénomène :** Dans ce cas, vous pourriez constater une charge système assez élevée, avec une valeur de la ```load average``` dans la commande ```status``` égale à 1 ou plus. Vous pouvez également voir que le nombre de ```exit_count``` du processus est élevé et continue d'augmenter.
**Résolution :** Exécutez workerman en mode débogage (sans l'option ```-d```) avec ```php start.php start```, puis recherchez les erreurs dans l'application et corrigez-les.

**2. Boucle infinie dans le code**
**Phénomène :** Vous pourriez voir un processus occupé consommant beaucoup de CPU dans la commande ```top```, et la commande ```strace -ttp pid``` ne donne aucune information sur les appels système.
**Résolution :** Consultez l'article de Birdman pour localiser le code PHP à l'aide de gdb et du code source de PHP, en suivant approximativement ces étapes :
1. Exécutez ```php -v``` pour connaître la version.
2. [Téléchargez le code source correspondant de PHP](https://www.php.net/releases/).
3. Exécutez ```gdb --pid=PID du processus occupé```.
4. Exécutez ```source chemin_du_code_source_PHP/.gdbinit```.
5. Exécutez ```zbacktrace``` pour afficher la pile d'appels.
Vous pourrez ainsi voir la pile d'appels de code PHP en cours d'exécution, c'est-à-dire l'emplacement de la boucle infinie. Remarque : Si ```zbacktrace``` ne donne pas de pile d'appels, il est possible que votre version de PHP n'ait pas été compilée avec l'option ```-g```, auquel cas vous devrez recompiler PHP avant de redémarrer workerman pour localiser le problème.

**3. Ajout infini de minuteries**
Si le code métier ajoute constamment des minuteries sans les supprimer, le processus finira par exécuter un nombre infini de minuteries, ce qui le maintiendra dans un état ```busy``` sans fin.
Exemple de code incorrect :
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
```
Dans ce code, lorsqu'un client se connecte, une minuterie est ajoutée, mais il n'y a aucune logique pour supprimer la minuterie dans l'ensemble du code métier. Ainsi, avec le temps, le nombre de minuteries dans le processus augmentera sans cesse, finissant par maintenir le processus dans un état ```busy``` infini. Le code correct serait :
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
```
