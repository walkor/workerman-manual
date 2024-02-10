# Notes
## Remarques sur l'utilisation des minuteries
1. Les minuteries ne peuvent être ajoutées qu'à l'intérieur des rappels ```onXXXX```. Il est recommandé de configurer les minuteries globales dans le rappel ```onWorkerStart``` et les minuteries pour une connexion spécifique dans le rappel ```onConnect```.

2. Les tâches planifiées ajoutées s'exécutent dans le processus actuel (sans démarrer de nouveaux processus ou threads). Si une tâche est très lourde (en particulier celles impliquant des opérations IO réseau), cela pourrait bloquer le processus et empêcher temporairement le traitement d'autres activités. Il est préférable d'exécuter les tâches chronophages dans un processus distinct, par exemple en créant un ou plusieurs processus Worker dédiés.

3. Lorsque le processus est occupé par d'autres activités ou si une tâche n'est pas complétée dans le délai prévu, elle attendra que la tâche en cours soit terminée avant de s'exécuter à nouveau au prochain intervalle. Cela peut entraîner un décalage dans l'exécution des minuteries par rapport à l'intervalle prévu. Autrement dit, les activités du processus actuel sont exécutées de manière séquentielle, mais dans un environnement multi-processus, les tâches s'exécutent en parallèle entre les processus.

4. Il est important de noter que la configuration de minuteries pour plusieurs processus peut entraîner des problèmes de concurrence. Par exemple, le code suivant imprime cinq fois par seconde dans chaque processus :
   ```php
   $worker = new Worker();
   // 5 processus
   $worker->count = 5;
   $worker->onWorkerStart = function(Worker $worker) {
       // 5 processus, chaque processus a une minuterie comme celle-ci
       Timer::add(1, function(){
           echo "hi\r\n";
       });
   };
   Worker::runAll();
   ```
   Si vous souhaitez qu'une seule instance de la minuterie s'exécute, veuillez consulter [Exemple 2 de Timer::add](add.md).

5. Il peut y avoir une erreur d'environ 1 milliseconde.

6. Il n'est pas possible de supprimer des minuteries entre les processus. Par exemple, une minuterie configurée dans le processus A ne peut pas être supprimée directement dans le processus B en utilisant l'interface Timer::del.

7. Les identifiants de minuteries peuvent se répéter entre les différents processus, mais les identifiants de minuteries générés dans le même processus ne se répéteront pas.

8. Le comportement des minuteries est affecté par un changement de l'heure système. Il est donc conseillé de redémarrer avec un restart après avoir modifié l'heure système.
