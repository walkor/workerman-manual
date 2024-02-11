# La classe Worker
Dans Workerman, il y a deux classes importantes : Worker et Connection.

La classe Worker est utilisée pour écouter les ports et peut définir des fonctions de rappel pour les événements de connexion client, les événements de message sur la connexion et les événements de déconnexion, afin de mettre en œuvre le traitement des activités. 

Il est possible de définir le nombre de processus de l'instance Worker (propriété count). Le processus principal de Worker va créer count sous-processus pour écouter le même port, recevoir les connexions des clients en parallèle et traiter les événements de connexion.
