# Ottimizzazione del kernel Linux

Per supportare una maggiore concorrenza nel sistema, oltre all'installazione dell'[estensione event](../install/install.md), è fondamentale ottimizzare il kernel Linux. Ogni singola ottimizzazione elencata di seguito è estremamente importante, pertanto si prega di completare ognuna di esse.

**Spiegazione dei parametri:**

> **max-file**: rappresenta il numero di gestori di file che possono essere aperti a livello di sistema. Questo valore si applica all'intero sistema operativo e non a singoli utenti.
> 
> **ulimit -n**: controlla il numero di gestori di file che possono essere aperti a livello di processo. Regola il numero di gestori di file disponibili per l'utente corrente della shell e per i processi avviati da esso.

Per verificare il numero di gestori di file che possono essere aperti a livello di sistema, eseguire il comando: `cat /proc/sys/fs/file-max`.

Aprire il file /etc/sysctl.conf e aggiungere le seguenti impostazioni:

```conf
# Imposta il numero massimo di TIME_WAIT del sistema. Verranno cancellati immediatamente se il valore supera quello predefinito.
net.ipv4.tcp_max_tw_buckets = 20000
# Definisce la lunghezza massima della coda di ascolto per ogni porta nel sistema.
net.core.somaxconn = 65535
# Numero massimo di connessioni TCP in attesa di conferma da parte della controparte che possono essere contenute nella coda.
net.ipv4.tcp_max_syn_backlog = 262144
# Massimo numero di pacchetti inviati alla coda quando la velocità di ricezione supera la velocità di elaborazione del kernel.
net.core.netdev_max_backlog = 30000
# Questa opzione può causare problemi di timeout per i client in una rete NAT. Raccomandato: 0
net.ipv4.tcp_tw_recycle = 0
# Numero massimo di file apribili da tutti i processi nel sistema
fs.file-max = 6815744
# Dimensione della tabella dei tracciamenti del firewall. Nota: Se il firewall non è attivo, verrà mostrato un errore "net.netfilter.nf_conntrack_max" è una chiave sconosciuta. È possibile ignorarlo.
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```

Eseguire il comando `sysctl -p` per applicare immediatamente le modifiche.

**Nota:**

Il file /etc/sysctl.conf offre numerose opzioni configurabili e altre impostazioni possono essere personalizzate in base alle esigenze dell'ambiente.

## Apertura dei file

Impostare il numero massimo di file aperti per risolvere il problema "too many open files" in situazioni ad alta concorrenza. Questa opzione influisce direttamente sul numero massimo di connessioni dei client che un singolo processo può gestire.

Il parametro "Soft open files" è un parametro di sistema Linux che influenza il numero massimo di gestori di file che un singolo processo può aprire. Questo valore influisce sul numero massimo di connessioni utente che un singolo processo può gestire in applicazioni a connessione persistente, come le chat. Eseguendo il comando `ulimit -n`, è possibile visualizzare il valore di questo parametro. Ad esempio, se è impostato su 1024, significa che un singolo processo può gestire al massimo 1024 connessioni utente contemporaneamente (o anche meno, visto che ci sono altri file aperti). Se vengono avviati 4 processi per gestire le connessioni utente, il numero massimo di connessioni contemporanee supportate dall'applicazione non supererà 4*1024. Aumentare questa impostazione consente all'applicazione di gestire un maggior numero di connessioni TCP.

**Tre metodi per modificare Soft open files:**

Primo metodo: eseguire direttamente in terminale `ulimit -HSn 102400` e riavviare workerman. Questo vale solo per il terminale corrente e il valore di open files tornerà al valore predefinito dopo la disconnessione.

Secondo metodo: aggiungere `ulimit -HSn 102400` alla fine del file `/etc/profile`, in modo che venga eseguito automaticamente ogni volta che si effettua l'accesso al terminale. Dopo la modifica è necessario riavviare workerman.

Terzo metodo: per rendere la modifica del valore di open files permanente, è necessario modificare il file di configurazione: `/etc/security/limits.conf`. Aggiungere quanto segue alla fine del file:

```
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

Questa modalità richiede il riavvio del server per entrare in vigore.
