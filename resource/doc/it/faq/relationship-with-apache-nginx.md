# Relazione con Apache e Nginx

**Domanda:**

Qual è la relazione tra Workerman e Apache/nginx/php-fpm? Ci sono conflitti tra Workerman e Apache/nginx/php-fpm?

**Risposta:**

Workerman e Apache/nginx/php-fpm non hanno nessuna relazione e il funzionamento di Workerman non dipende da Apache/nginx/php-fpm. Sono entrambi contenitori indipendenti, non interferiscono tra loro e non entrano in conflitto (a meno che non siano in ascolto sulla stessa porta).

Workerman è un framework server socket generico che supporta le connessioni persistenti e vari protocolli come HTTP, WebSocket e protocolli personalizzati. D'altra parte, Apache/nginx/php-fpm sono generalmente utilizzati solo per progetti Web che utilizzano il protocollo HTTP.

Se un server ha già installato Apache/nginx/php-fpm, l'installazione di Workerman non influirà sul loro funzionamento.
