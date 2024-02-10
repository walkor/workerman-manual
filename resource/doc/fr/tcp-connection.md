# Les interfaces fournies par la classe Connection

WorkerMan comprend deux classes importantes: Worker et Connection.

Chaque connexion client correspond à un objet Connection, avec des rappels tels que onMessage, onClose, etc. Il fournit également des interfaces telles que send pour envoyer des données au client et close pour fermer la connexion, ainsi que d'autres interfaces nécessaires.

On peut dire que Worker est un conteneur d'écoute, chargé d'accepter les connexions des clients et de les fournir sous forme d'objets de connexion aux développeurs pour les manipuler.
