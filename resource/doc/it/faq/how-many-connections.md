# Workerman supporta quanti utenti simultanei

Il concetto di **simultaneità** è ambiguo, qui verranno illustrati due indicatori quantificabili: il **numero di connessioni simultanee** e il **numero di richieste simultanee**.

Il **numero di connessioni simultanee** indica quante connessioni TCP il server sta mantenendo in un dato momento, anche se non ci sia alcuna comunicazione di dati su tali connessioni. Ad esempio, un server di push di notifiche potrebbe mantenere un milione di connessioni da dispositivi, ma poche di queste connessioni effettuano una comunicazione effettiva di dati, quindi il carico su questo server potrebbe essere quasi nullo, e potrebbe ancora accettare ulteriori connessioni fintanto che ci sia abbastanza memoria disponibile.

Il **numero di richieste simultanee** viene generalmente misurato in QPS (quante richieste il server gestisce al secondo), e non si interessa tanto del numero di connessioni TCP attive sul server in un dato momento. Ad esempio, un server potrebbe avere solo 10 connessioni client, ma ognuna di esse riceve 10.000 richieste al secondo, quindi il server deve essere in grado di sostenere almeno 10 * 10.000 = 100.000 richieste al secondo. Supponendo che 100.000 richieste al secondo sia il limite massimo di questo server, se ogni client invia solo una richiesta al secondo al server, allora il server può sostenere 100.000 client.

Il **numero di connessioni simultanee** è limitato dalla memoria del server, e un server Workerman con 24GB di memoria può supportare approssimativamente **1.200.000** connessioni simultanee.

Il **numero di richieste simultanee** è limitato dalla capacità di elaborazione della CPU del server e una singola macchina con 24 core Workerman può raggiungere **450.000** richieste al secondo (QPS), con valori effettivi che variano in base alla complessità del servizio e alla qualità del codice.

## Nota

Per scenari ad alta simultaneità è necessario installare l'estensione event, fare riferimento al capitolo sull'installazione e configurazione. Inoltre è necessario ottimizzare il kernel Linux, in particolare il limite del numero di file aperti per processo, fare riferimento al capitolo sull'ottimizzazione del kernel nell'appendice.

## Dati dello stress test
> I dati provengono da un'ente autorevole di test di stress di terze parti techempower.com 20° round di test di stress
https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf

**Configurazione del server:**
Core totali 14, Thread totali 28, 32 GB di memoria, switch Ethernet Cisco dedicato da 10 gigabit
**Logica commerciale:**
Attività con query al database, database pgsql, PHP8+JIT
QPS superiore a 390.000
![](../images/screenshot_1636522357217.png)
