# Creare un servizio wss con Workerman

**Domanda:**

Come posso creare un servizio di wss con Workerman in modo che i client possano comunicare tramite la connessione wss, ad esempio in un'applicazione WeChat Mini Program?

**Risposta:**

Il protocollo wss è effettivamente una combinazione di [websocket](https://baike.baidu.com/item/WebSocket) e [SSL](https://baike.baidu.com/item/ssl), ovvero un'aggiunta dello strato [SSL](https://baike.baidu.com/item/ssl) al protocollo websocket, simile a [https](https://baike.baidu.com/item/https) ([http](https://baike.baidu.com/item/http) + [SSL](https://baike.baidu.com/item/ssl)). Pertanto, è sufficiente abilitare [SSL](https://baike.baidu.com/item/ssl) sul protocollo [websocket](https://baike.baidu.com/item/WebSocket) base per supportare il protocollo wss.

## Metodo 1: Utilizzo di nginx/apache come proxy SSL (raccomandato)

**Principio e flusso di comunicazione**

1. Il client avvia la connessione wss a nginx/apache
2. Nginx/apache converte i dati del protocollo wss in dati del protocollo ws e li inoltra alla porta del protocollo websocket di Workerman
3. Workerman riceve i dati e li elabora logicamente
4. Quando Workerman invia un messaggio al client, si verifica il processo inverso, i dati vengono convertiti in protocollo wss tramite nginx/apache e inviati al client

## Esempio di configurazione nginx

**Requisiti e preparativi:**

1. Nginx installato, versione non inferiore a 1.3
2. Si presume che Workerman stia ascoltando sulla porta 8282 (protocollo websocket)
3. È stato ottenuto un certificato (file pem/crt e file chiave) e si presume che siano posizionati in /etc/nginx/conf.d/ssl
4. Intenzione di utilizzare nginx per aprire la porta 443 per fornire un servizio proxy wss all'esterno (la porta può essere modificata secondo necessità)
5. Nginx viene solitamente eseguito come server web per altri servizi. Per non influenzare l'uso del sito originale, qui viene utilizzato l'indirizzo `dominio.com/wss` come punto di ingresso per il proxy wss. In altre parole, l'indirizzo di connessione del cliente è wss://domain.com/wss

**Configurazione di nginx:**

```nginx
server {
  listen 443;
  # Configurazione del dominio omit...
  
  ssl on;
  ssl_certificate /etc/ssl/server.pem;
  ssl_certificate_key /etc/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location /wss
  {
    proxy_pass http://127.0.0.1:8282;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header X-Real-IP $remote_addr;
  }
  
  # Altre configurazioni del sito...
}
```

**Test:**

```javascript
// Il certificato è controllato dal dominio. Utilizzare l'indirizzo del dominio. Attenzione: non specificare la porta qui
ws = new WebSocket("wss://dominio.com/wss");

ws.onopen = function() {
    alert("Connessione riuscita");
    ws.send('tom');
    alert("Inviato al server una stringa: tom");
};
ws.onmessage = function(e) {
    alert("Ricevuto il messaggio dal server: " + e.data);
};
```

## Utilizzo di Apache come proxy wss

È anche possibile utilizzare Apache come proxy per inoltrare wss a Workerman.

**Preparativi:**

1. GatewayWorker ascolta sulla porta 8282 (protocollo websocket)
2. È stato ottenuto il certificato SSL, presumibilmente posizionato in /server/httpd/cert/
3. Apache inoltra il traffico dalla porta 443 alla porta 8282 specifica
4. Il file httpd-ssl.conf è stato caricato
5. OpenSSL è installato

**Abilitare il modulo proxy_wstunnel_module**

```apache
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
```

**Configurazione SSL e proxy**

```apache
#extra/httpd-ssl.conf
DocumentRoot "/percorso/sito"
ServerName dominio

# Configurazioni del proxy
SSLProxyEngine on

ProxyRequests Off
ProxyPass /wss ws://127.0.0.1:8282/wss
ProxyPassReverse /wss ws://127.0.0.1:8282/wss

# Aggiunta del supporto ai protocolli SSL, rimuovendo i protocolli non sicuri
SSLProtocol all -SSLv2 -SSLv3
# Modifica le suite crittografiche come segue
SSLCipherSuite HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM
SSLHonorCipherOrder on
# Configurazioni del certificato pubblico
SSLCertificateFile /server/httpd/cert/your.pem
# Configurazioni della chiave privata
SSLCertificateKeyFile /server/httpd/cert/your.key
# Configurazioni della catena di certificati
SSLCertificateChainFile /server/httpd/cert/chain.pem
```

**Test:**

```javascript
// Il certificato è controllato dal dominio. Utilizzare l'indirizzo del dominio. Attenzione: non specificare la porta qui
ws = new WebSocket("wss://dominio.com/wss");

ws.onopen = function() {
    alert("Connessione riuscita");
    ws.send('tom');
    alert("Inviato al server una stringa: tom");
};
ws.onmessage = function(e) {
    alert("Ricevuto il messaggio dal server: " + e.data);
};
```

## Metodo 2, abilita SSL con Workerman direttamente (non raccomandato)

> **Nota:**
> È possibile abilitare SSL con nginx/apache o con Workerman direttamente, ma non entrambi contemporaneamente.

**Preparativi:**

1. Versione di Workerman >= 3.3.7
2. PHP con estensione openssl installata
3. Certificato ottenuto (file pem/crt e file chiave) posizionato in qualsiasi directory sul disco

**Codice:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// È meglio utilizzare un certificato ottenuto da una CA
$context = array(
    // Per ulteriori opzioni SSL, consultare il manuale http://php.net/manual/zh/context.ssl.php
    'ssl' => array(
        // Utilizzare un percorso assoluto
        'local_cert'        => 'percorso_su_disco/server.pem', // Può anche essere un file crt
        'local_pk'          => 'percorso_su_disco/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // Necessario per i certificati autofirmati
    )
);
// Configurazione del protocollo websocket (la porta può essere qualsiasi, ma assicurarsi che non sia utilizzata da altri programmi)
$worker = new Worker('websocket://0.0.0.0:8282', $context);
// Abilita SSL per il trasporto, websocket+ssl corrisponde a wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Con il codice sopra, Workerman sta ascoltando il protocollo wss e i client possono connettersi usando il protocollo wss per realizzare la comunicazione istantanea in modo sicuro.

**Test:**

Aprire il browser Chrome, premere F12 per aprire la console di debug, quindi nella scheda Console inserire il seguente codice JavaScript (o inserire il seguente codice HTML e farlo eseguire tramite JavaScript):

```javascript
// Il certificato è controllato dal dominio. Utilizzare l'indirizzo del dominio. Attenzione: usare la porta qui
ws = new WebSocket("wss://dominio.com:8282");
ws.onopen = function() {
    alert("Connessione riuscita");
    ws.send('tom');
    alert("Inviato al server una stringa: tom");
};
ws.onmessage = function(e) {
    alert("Ricevuto il messaggio dal server: " + e.data);
};
```

**Nota:**

1. Se è necessario utilizzare la porta 443, utilizzare il primo metodo di proxy nginx/apache per implementare il wss.
2. La porta wss può essere acceduta solo tramite il protocollo wss; il protocollo ws non può accedere alla porta wss.
3. In genere, i certificati sono associati a un dominio, quindi, durante il test, utilizzare l'indirizzo del dominio per la connessione e non l'indirizzo IP.
4. Se si verificano problemi di accesso, controllare il firewall del server.
5. Questo metodo richiede che la versione di PHP sia >=5.6, poiché le Mini App di WeChat richiedono tls1.2 e le versioni di PHP inferiori a 5.6 non supportano tls1.2.

Articoli correlati:  
[Ottenere l'indirizzo IP reale tramite un proxy](get-real-ip-from-proxy.md)  
[Riferimento alle opzioni del contesto SSL di Workerman](https://php.net/manual/zh/context.ssl.php)
