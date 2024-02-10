# Prefazione

**Workerman, un contenitore di applicazioni PHP ad alte prestazioni**

## Che cos'è Workerman?
Workerman è un contenitore di applicazioni PHP ad alte prestazioni open source sviluppato interamente in PHP.

Workerman non è un reinventare la ruota, non è un framework MVC, ma è un framework di servizio più generico e di livello inferiore. Puoi usarlo per sviluppare un proxy TCP, un proxy per VPN, server di giochi, server di posta, server FTP e persino sviluppare una versione PHP di Redis, una versione PHP di un database, una versione PHP di Nginx, una versione PHP di PHP-FPM e molto altro. Workerman può essere considerato un'innovazione nel campo PHP, che libera completamente gli sviluppatori dalla restrizione che PHP può solo gestire applicazioni Web.

In realtà, Workerman è simile a una versione di PHP di Nginx, con il nucleo basato su multiprocessi, Epoll e I/O non bloccante. Ogni processo di Workerman può gestire decine di migliaia di connessioni simultanee. Poiché risiede in memoria costante e non dipende da Apache, Nginx, PHP-FPM e simili, possiede un'elevata efficienza. Supporta anche TCP, UDP, UNIX SOCKET, connessioni persistenti, Websocket, HTTP, WSS, HTTPS e vari protocolli personalizzati. Inoltre, dispone di numerosi componenti ad alte prestazioni come timer, client socket asincrono, Redis asincrono, HTTP asincrono, code di messaggi asincrone e altro.

## Alcune direzioni di utilizzo di Workerman
Workerman, a differenza dei tradizionali framework MVC, può essere utilizzato non solo per lo sviluppo Web, ma anche in campi molto più ampi come la comunicazione istantanea, l'Internet delle cose, i giochi, la governace dei servizi e altri server o middleware, aumentando considerevolmente la visibilità degli sviluppatori PHP. Attualmente, gli sviluppatori PHP in questi settori sono ancora relativamente pochi. Se desideri avere un vantaggio tecnico nel campo PHP e non ti accontenti di operazioni CRUD quotidiane, o desideri progredire verso il ruolo di architetto o esperto tecnico, Workerman è un framework molto valido da imparare. Si consiglia agli sviluppatori di non solo saper usare Workerman, ma anche di sviluppare progetti open source basati su Workerman, aumentando così le proprie competenze e la propria influenza. Ad esempio, [Beanbun, un framework multiprocesso per web crawling](https://github.com/kiddyuchina/Beanbun), è un ottimo esempio che ha ricevuto molti apprezzamenti appena è stato lanciato.

Alcune direzioni di utilizzo di Workerman includono:

1. Comunicazione istantanea
Ad esempio, chat istantanea su pagine web, notifiche di messaggi istantanei, app di messaggistica WeChat, notifiche di messaggi per app mobili e software per PC.
[[Esempi chat room workerman-chat](https://www.workerman.net/workerman-chat), [web push notifications](https://www.workerman.net/web-sender), [chat room "Todpole"](https://www.workerman.net/workerman-todpole)]

2. Internet delle cose
Ad esempio, comunicazione con stampanti, microcontrollori, smartband, domotica, bike sharing e altro.
[Esempi di clienti come "Yilianyun", "Easy Parking Time" e altri]

3. Server di gioco
Ad esempio, giochi da tavolo, giochi MMORPG e altro.
[Browserquest-php](https://www.workerman.net/browserquest)

4. Servizi HTTP
Ad esempio, sviluppo di interfacce HTTP ad alte prestazioni e siti web ad alte prestazioni. Se desideri creare servizi o siti web HTTP, è fortemente raccomandato [webman](https://github.com/walkor/webman).

5. SOA (Service-Oriented Architecture)
Utilizzando Workerman per incapsulare le diverse unità funzionali di un'attività esistente in un servizio che fornisce un'interfaccia unificata all'esterno, è possibile raggiungere l'accoppiamento lasco del sistema, una facile manutenzione, alta disponibilità e scalabilità. Esempi includono [workerman-json-rpc](https://github.com/walkor/workerman-jsonrpc) e [workerman-thrift](https://github.com/walkor/workerman-thrift).

6. Altri software server
Ad esempio, [GatewayWorker](https://www.workerman.net/doc/gateway-worker), [PHPSocket.IO](https://www.workerman.net/phpsocket_io), server proxy HTTP, proxy socks5, componente di comunicazione distribuita, componente di condivisione di variabili distribuite, code di messaggi, server DNS, server Web, server CDN, server FTP e altro.

7. Componenti
Come [Redis asincrono](components/workerman-redis.md), [client HTTP asincrono](components/workerman-http-client.md), [client Mqtt per Internet delle cose](components/workerman-mqtt.md), code di messaggi [workerman/redis-queue](components/workerman-redis-queue.md), [workerman/stomp](components/workerman-stomp.md), [workerman/rabbitmq](components/workerman-rabbitmq.md), componente di monitoraggio dei file, nonché molti framework e componenti sviluppati da terze parti e altro.

Chiaramente, i tradizionali framework MVC sono difficili da utilizzare per realizzare le funzionalità sopra citate, motivo per cui è nato Workerman.

## Filosofia di Workerman
Semplicità, stabilità, alte prestazioni e distribuzione.

### **Semplicità**
Il motto "piccolo è bello" si adatta bene a Workerman. Il nucleo di Workerman è estremamente semplice, costituito da pochi file PHP e con poche API esposte, rendendo l'apprendimento molto semplice. Tutte le altre funzionalità vengono estese tramite componenti.

Workerman ha una documentazione completa, un sito web autorevole, una comunità attiva, diversi gruppi QQ con migliaia di membri e molti componenti ad alte prestazioni e numerosi esempi, tutto ciò rende l'utilizzo da parte degli sviluppatori estremamente facile.

### **Stabilità**
Workerman è stato open source per diversi anni ed è stato ampiamente utilizzato da numerose grandi aziende, garantendone la grande stabilità. Alcuni servizi non sono stati riavviati per oltre 2 anni e continuano a funzionare rapidamente. Non ci sono core dump, memory leak o bug.

### **Alte prestazioni**
Grazie alla residenza costante in memoria e alla mancanza di dipendenza da Apache, Nginx, PHP-FPM e simili, Workerman non ha alcun overhead di comunicazione tra i container e PHP, né il costo di inizializzazione e distruzione per ogni richiesta, garantendo alte prestazioni. Rispetto ai tradizionali framework MVC, le prestazioni possono essere decine di volte superiori. Sotto PHP7, i test di stress con ApacheBench hanno mostrato QPS superiori persino a Nginx singolo.

### **Distribuzione**
Nell'era attuale, non è più sufficiente essere indipendenti. Anche un server singolo ha i suoi limiti. La distribuzione su più server è la chiave del successo. Workerman fornisce direttamente una soluzione di comunicazione distribuita per le connessioni persistenti con il framework [GatewayWorker](https://doc2.workerman.net). È sufficiente aggiungere un server e avviare, senza alcuna modifica al codice di business, ottenendo un'espansione del sistema multipla. Se stai sviluppando un'applicazione di connessione persistente TCP, ti consigliamo di utilizzare direttamente [GatewayWorker](https://doc2.workerman.net), che è un'ulteriore integrazione di Workerman con un'interfaccia più ricca e un notevole potenziale di distribuzione.

## Ambito di applicazione di questo manuale
Versioni da Workerman 3.x a 4.x

## Utenti Windows (letto obbligatorio)
Workerman supporta sia i sistemi Linux che Windows. La versione Windows di Workerman **non dipende da alcuna estensione**, richiede solo la configurazione corretta delle variabili di ambiente di PHP. Per ulteriori informazioni sull'installazione e le istruzioni importanti per la versione Windows di Workerman, vedere [Guide per gli utenti di Windows (in inglese)](https://www.workerman.net/windows).

## Client
Il protocollo di comunicazione di Workerman è aperto e personalizzabile. Di conseguenza, in teoria Workerman può comunicare con client di qualsiasi piattaforma che utilizzano qualsiasi tipo di protocollo. Quando si sviluppa un client, è possibile completare la comunicazione con il server in base al protocollo di comunicazione corrispondente.


