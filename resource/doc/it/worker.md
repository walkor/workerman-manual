# Classe Worker
In Workerman ci sono due classi importanti: Worker e Connection.

La classe Worker viene utilizzata per ascoltare le porte e può essere configurata con le funzioni di callback per gli eventi di connessione del client, di ricezione del messaggio dalla connessione e di disconnessione della connessione, in modo da gestire le attività di business.

È possibile impostare il numero di processi dell'istanza Worker (attraverso l'attributo count). Il processo principale di Worker creerà count processi figlio che ascolteranno contemporaneamente la stessa porta, in modo da gestire in parallelo le connessioni dei client e i relativi eventi.
