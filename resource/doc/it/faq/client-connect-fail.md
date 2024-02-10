# Motivi dei fallimenti della connessione del cliente

Di solito ci sono due tipi di errori di connessione falliti del cliente, ```connection refuse``` e ```connection timeout```

## Connection Refuse (Connessione rifiutata)

Di solito è dovuto a:
1. Il cliente ha sbagliato la porta di connessione
2. Il cliente ha sbagliato il nome di dominio o l'indirizzo IP di connessione
3. Se il cliente utilizza un nome di dominio per la connessione, potrebbe puntare a un IP server errato
4. Il server utilizza CDN o altri proxy di accelerazione che causano un IP effettivo di connessione diverso dall'IP previsto
5. Il server non è avviato o la porta non è in ascolto
6. Utilizzo di software proxy di rete
7. L'IP di ascolto del server e l'indirizzo di accesso non sono nello stesso intervallo di indirizzi. Ad esempio, se il server ascolta su 127.0.0.1, il cliente può solo connettersi tramite 127.0.0.1, non tramite l'IP di rete locale o l'IP pubblico. Si consiglia di impostare l'indirizzo di ascolto su 0.0.0.0 in modo che il computer locale, la rete locale e la rete pubblica possano connettersi.

## Connection Timeout (Connessione scaduta)

Di solito è dovuto a:
1. Il firewall del server blocca la connessione, è possibile disattivare temporaneamente il firewall e provare
2. Se si tratta di un server cloud, il gruppo di sicurezza potrebbe impedire l'apertura della porta corrispondente, è necessario aprire la porta corrispondente nel pannello di amministrazione
3. Se si utilizza un pannello di controllo come Plesk, è necessario aprire la porta corrispondente in Plesk
4. Il server non esiste o non è avviato
5. Se il cliente utilizza un nome di dominio per la connessione, potrebbe puntare a un IP server errato
6. L'IP a cui il cliente sta tentando di accedere è l'IP interno del server e il cliente e il server non si trovano nella stessa rete locale

## Cannot assign requested address (Impossibile assegnare l'indirizzo richiesto)

**Come cliente**, ogni volta che viene avviata una connessione, è necessario utilizzare una porta temporanea locale. Di default, un server dispone di un numero di porte temporanee disponibili che si aggira intorno a 20-30 mila. Se il numero di connessioni avviate verso un server specifico supera questo valore, non sarà possibile assegnare ulteriori porte disponibili, generando questo errore.
Si può aumentare il numero di porte temporanee locali modificando il parametro del kernel `/etc/sysctl.conf` `net.ipv4.ip_local_port_range`, ad esempio impostandolo su `10000 65535` (aumentando così il numero di porte locali disponibili a 55535), quindi eseguire il comando `sysctl -p` per rendere effettive le modifiche.
Inoltre, quando una connessione viene interrotta, rimarrà in stato TIME_WAIT e continuerà a occupare la porta locale corrispondente per un certo periodo di tempo. Questo significa che anche un grande numero di connessioni brevi (superiore a 20-30 mila) in un breve lasso di tempo potrebbe generare il messaggio di errore `Cannot assign requested address`. In questo caso, è possibile risolvere il problema abilitando il rapido recupero delle connessioni TIME_WAIT nel kernel. Per ulteriori dettagli, fare riferimento a [Tuning del kernel](https://www.workerman.net/doc/workerman/appendices/kernel-optimization.html).

> **Nota**
> Il numero delle porte locali è limitato solo per il cliente, il server non ha alcun limite sulle porte locali, purché le risorse siano sufficienti, il server può mantenere un numero illimitato di connessioni.

## Altri errori
Se si verifica un errore diverso da ```connection refuse``` e ```connection timeout```, di solito è dovuto a:

**1. Il protocollo di comunicazione utilizzato dal cliente e dal server non corrisponde.**
Ad esempio, se il server utilizza il protocollo di comunicazione HTTP, il cliente non può connettersi utilizzando il protocollo di comunicazione WebSocket. Se il cliente utilizza il protocollo WebSocket per la connessione, il server deve essere anche con il protocollo WebSocket. Se il server fornisce un servizio con il protocollo HTTP, allora il cliente deve utilizzare il protocollo HTTP per la connessione.

Il principio è simile a dover utilizzare l'inglese per comunicare con una persona inglese o utilizzare il giapponese per comunicare con una persona giapponese. In questo caso, il linguaggio è simile al protocollo di comunicazione, entrambe le parti (cliente e server) devono utilizzare lo stesso linguaggio per comunicare, altrimenti non sarà possibile la comunicazione.

**Errori comuni causati da protocolli di comunicazione non corrispondenti:**

> Connessione WebSocket a 'ws://xxx.com:xx/' fallita: Errore durante l'handshake WebSocket: Codice di risposta inaspettato: xxx

> Connessione WebSocket a 'ws://xxx.com:xx/' fallita: Errore durante l'handshake WebSocket: net::ERR_INVALID_HTTP_RESPONSE

**Soluzione:**
Dalle due descrizioni di errore sopra, si può vedere che il cliente sta utilizzando una connessione ws con protocollo WebSocket. Il server deve essere configurato con il protocollo WebSocket per permettere la connessione, ad esempio:

Se si tratta di gatewayWorker, il codice di ascolto sarà simile a
```php
// Protocollo WebSocket, in questo modo il cliente può connettersi con ws://... Porta xxx da mantenere
$gateway = new Gateway('websocket://0.0.0.0:xxxx');
```
Se si tratta di Workerman, sarà simile a
```php
// Protocollo WebSocket, in questo modo il cliente può connettersi con ws://... Porta xxx da mantenere
$worker = new Worker('websocket://0.0.0.0:xxxx');
```
