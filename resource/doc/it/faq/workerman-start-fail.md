# Avvio fallito di Workerman

## Fenomeno 1
Dopo l'avvio, viene visualizzato un errore simile a quanto segue:
```php
php start.php start
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (Indirizzo già in uso) in ...workerman/Worker.php on line xxxx
```
**Parole chiave**: ```Indirizzo già in uso```

**Cause principali**: La porta è già occupata e non può essere avviata.

#### Soluzione 1
È possibile individuare il programma che sta utilizzando la porta mediante il comando ```netstat -anp | grep numero_porta```, e quindi arrestarlo per liberare la porta.

#### Soluzione 2
Se non è possibile arrestare il programma corrispondente alla porta, è possibile risolvere la situazione cambiando la porta di Workerman.

#### Soluzione 3
Se la porta è occupata da Workerman e non può essere arrestata con il comando "stop" (solitamente a causa della perdita del file pid o dell'uccisione del processo principale da parte dello sviluppatore), è possibile uccidere il processo Workerman eseguendo i seguenti due comandi.

```bash
killall php
ps aux|grep WorkerMan|awk '{print $2}'|xargs kill -9
```

#### Soluzione 4
Se effettivamente nessun programma sta ascoltando su quella porta, potrebbe essere che lo sviluppatore abbia configurato più di un ascolto in Workerman con la stessa porta. Si invita lo sviluppatore a verificare se lo script di avvio sta effettuando più di un ascolto sulla stessa porta.

#### Soluzione 5
Verificare se il programma ha attivato il riutilizzo della porta (reusePort). Provare a disattivare il riutilizzo della porta.

## Fenomeno 2
Dopo l'avvio, viene visualizzato un errore simile a quanto segue:
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxx (Impossibile assegnare l'indirizzo richiesto) in ...workerman/Worker.php on line xxxx
```
oppure
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://xx.xx.xx.xx:xxxx (L'indirizzo richiesto non è valido nel suo contesto) in ...workerman/Worker.php on line xxxx
```
**Parole chiave**: `Impossibile assegnare l'indirizzo richiesto` o `L'indirizzo richiesto non è valido`

**Cause del fallimento**: Errore di scrittura del parametro IP di ascolto nello script di avvio, l'indirizzo non è quello della macchina locale. È necessario inserire l'indirizzo IP della macchina o utilizzare ```0.0.0.0``` (che indica l'ascolto su tutte le interfacce di rete locali) per risolvere il problema.

**Consiglio**: Nel sistema Linux è possibile utilizzare il comando ```ifconfig``` per visualizzare tutti gli indirizzi IP delle schede di rete. Se si utilizza un server cloud (come Alibaba Cloud o Tencent Cloud), è importante notare che l'indirizzo IP pubblico potrebbe essere un indirizzo IP proxy e non quello effettivo del server, pertanto non sarà possibile effettuare l'ascolto su tale indirizzo. Tuttavia, è comunque possibile effettuare l'ascolto su tutte le interfacce di rete locali mediante l'utilizzo di ```0.0.0.0```.

## Fenomeno 3
```php
Attenzione: la funzione stream_socket_server è stata disabilitata per motivi di sicurezza in ...
```
**Cause del fallimento**: La funzione stream_socket_server è disabilitata nel file php.ini.

**Soluzione**: 
1. Eseguire il comando ```php --ini``` per individuare il file php.ini.
2. Aprire il file php.ini, individuare la voce disable_functions e rimuovere l'elemento di disabilitazione di stream_socket_server.

## Fenomeno 4
```php
PHP Warning:  stream_socket_server(): unable to connect to tcp://0.0.0.0:xxx (Permesso negato)
```
**Cause del fallimento**: Su Linux, se si ascolta su una porta inferiore a 1024, è necessario disporre dei privilegi di amministratore (root).

**Soluzione**: Utilizzare una porta superiore a 1024 oppure avviare il servizio con privilegi di root.
