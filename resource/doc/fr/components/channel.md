# Composant de communication distribué Channel
**``` (Workerman version requise >= 3.3.0) ```**

Adresse du code source : https://github.com/walkor/Channel

Channel est un composant de communication distribué utilisé pour la communication inter-processus ou inter-serveurs.

## Caractéristiques
1. Basé sur le modèle de publication/abonnement
2. Entrées/sorties non bloquantes

## Principe
Channel comprend un serveur Channel/Server et un client Channel/Client.
Le client Channel/Client se connecte au serveur Channel/Server en utilisant l'interface connect et maintient une connexion longue.
Le client Channel/Client informe le serveur Channel/Server des événements qu'il surveille en appelant l'interface on et en enregistrant les fonctions de rappel des événements (le rappel se produit dans le processus du client Channel/Client).
Le client Channel/Client publie un événement et les données associées au serveur Channel/Server via l'interface publish.
Le serveur Channel/Server reçoit l'événement et les données, puis les distribue au client Channel/Client qui surveille cet événement.
Le client Channel/Client, lorsqu'il reçoit l'événement et les données, déclenche le rappel défini par l'interface on.
Le client Channel/Client ne recevra que les événements auxquels il est abonné et déclenchera les rappels associés.

## Installation
`composer require workerman/channel`

## Remarque
Channel ne peut être utilisé que dans un environnement Workerman, il ne peut pas être utilisé dans un environnement PHP-FPM.
