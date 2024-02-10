# Raisons de l'échec de l'envoi dans la commande "status"

**Phénomène :**

Lors de l'exécution de la commande "status", on constate des cas d'échec d'envoi (send_fail). Quelles en sont les raisons ?

**Réponse :**

En général, la présence de send_fail n'est pas un problème majeur. Cela est généralement dû à la fermeture volontaire de la connexion par le client ou à l'incapacité du client à recevoir des données, ce qui entraîne l'échec de l'envoi des données.

Il y a deux raisons à l'échec de l'envoi (send_fail) :

1. Lorsque l'appel à l'interface send pour envoyer des données au client détecte que le client s'est déjà déconnecté, le compteur send_fail est incrémenté. Étant donné qu'il s'agit d'une déconnexion volontaire du client, il s'agit d'un phénomène normal et peut généralement être négligé.

2. La vitesse d'envoi de données du serveur est supérieure à la vitesse de réception du client, ce qui entraîne une accumulation continue de données dans le tampon de l'envoi côté serveur (Workerman crée un tampon d'envoi pour chaque client). Si la taille du tampon dépasse la limite (TcpConnection::$maxSendBufferSize par défaut à 1 Mo), les données seront jetées, déclenchant ainsi l'événement onError (le cas échéant) et entraînant une incrémentation du compteur send_fail.

Par exemple, lorsque le navigateur est réduit au minimum, le JavaScript peut cesser de s'exécuter, ce qui empêche la réception des données côté client, et celles-ci s'accumulent dans le tampon, dépassant la limite. Dans ce cas, chaque appel à send entraînera une incrémentation du compteur send_fail.

**Conclusion :**

En général, l'échec de l'envoi ("send_fail") dû à la déconnexion du client ne nécessite pas de préoccupations particulières.

Si l'échec de l'envoi est causé par l'arrêt de la réception des données par le client, il est nécessaire de vérifier si le client fonctionne correctement.

Si la vitesse de réception des données côté client est **continuellement** inférieure à la vitesse d'envoi côté serveur, il est nécessaire de considérer l'optimisation du flux de production, ou l'optimisation des performances du client. Si le problème est lié à la bande passante entraînant une faible fluidité de l'envoi, il peut être envisagé d'augmenter la bande passante du serveur.
