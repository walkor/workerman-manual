# Debugging a Busy Process
A volte attraverso il comando ```php start.php status``` possiamo vedere un processo in stato ```busy```, indicando che il processo corrispondente sta gestendo il business. Normalmente, dopo aver completato il business, il processo corrispondente tornerà allo stato di ```idle```, il che di solito non dovrebbe creare problemi. Tuttavia, se il processo rimane costantemente in stato ```busy``` senza tornare mai allo stato ```idle```, allora ciò indica che il business all'interno del processo è bloccato o in un ciclo infinito. In tal caso, è possibile individuarlo utilizzando i seguenti metodi.

## Utilizzo del comando strace+lsof per individuare

**1. Trovare il pid del processo in stato busy in status**
Eseguire il comando ```php start.php status``` mostra quanto segue:
![](../images/d1903ed65ef2f3b0850e84ccbedc52aa.png)
Nell'immagine, il ```pid``` del processo ```busy``` è ```11725``` e ```11748```.

**2. Tracciamento del processo con strace**
Scegliere un ```pid``` del processo (qui si sceglie ```11725```) e eseguire il comando ```strace -ttp 11725``` mostra quanto segue:
![](../images/7ce9f36da926f670949609dcdc593ab4.png)
Si può vedere che il processo sta continuamente eseguendo la chiamata di sistema ```poll([{fd=16, events=....```, che aspetta l'evento di lettura per il descrittore fd 16, ovvero attende il dato restituito da questo descrittore.

Se non viene visualizzata alcuna chiamata di sistema, mantenere il terminale attuale aperto, aprire un nuovo terminale e eseguire il comando ```kill -SIGALRM 11725``` (invia un segnale di sveglia al processo) e verificare se c'è una risposta nel terminale di strace e se è bloccato su una particolare chiamata di sistema. Se non viene ancora visualizzata alcuna chiamata di sistema, significa molto probabilmente che il programma è intrappolato in un ciclo di business, fare riferimento all'elemento 2 nella sezione finale di questa pagina per risolvere il problema.

Se il sistema è bloccato su una chiamata di sistema come epoll_wait o select, questa è una situazione normale e indica che il processo è in stato ```idle```.

**3. Visualizzazione dei descrittori di processo con lsof**
Eseguire il comando ```lsof -nPp 11725``` mostra quanto segue:
![](../images/27bd629c3a1ac93f9f4b535d01df2ac1.png)
Il descrittore 16 corrisponde al record 16u (ultima riga), e si può vedere che il descrittore ```fd=16``` corrisponde a una connessione TCP, con l'indirizzo remoto come ```101.37.136.135:80```: ciò indica che il processo sta probabilmente accedendo a una risorsa HTTP e il ciclo di ```poll([{fd=16, events=....``` continua ad attendere la risposta dal server HTTP, spiegando così il motivo per cui il processo è in stato ```busy```.

**Soluzione:**
Una volta individuato dov'è bloccato il processo, risolverlo diventa più facile. Ad esempio, nel caso individuato sopra, si presume che il business stia chiamando curl e la relativa URL non restituisce dati per molto tempo, causando il processo ad attendere costantemente. In questa situazione, è possibile contattare il fornitore dell'URL per individuarne la lentezza della risposta e nello stesso tempo, aggiungere un parametro di timeout durante la chiamata di curl. Ad esempio, se non riceve risposta entro 2 secondi, termina il processo di attesa, evitando così il blocco prolungato (in questo caso il processo potrebbe diventare ```busy``` per circa 2 secondi).

## Altre cause del processo in stato busy per un lungo periodo
Oltre al blocco del processo o altri motivi che causano il processo in stato ```busy```, ci sono le seguenti cause.

**1. Errore fatale nell'attività che causa l'uscita continua del processo**
**Sintomo:** In questa situazione, si può notare un carico di sistema piuttosto elevato, con il ```load average``` in ```status``` uguale o superiore a 1. Si potrà vedere che il numero di ```exit_count``` del processo è molto alto e continua ad aumentare.
**Soluzione:** Eseguire workerman in modalità di debug (senza l'opzione ```-d```) tramite il comando ```php start.php start``` e verificare gli errori nel business per risolverli.

**2. Ciclo infinito nel codice**
**Sintomo:** Nell'utility ```top``` si può vedere che il processo in stato ```busy``` sta occupando molto CPU, ma il comando ```strace -ttp pid``` non stampa alcuna informazione sulla chiamata di sistema.
**Soluzione:** Fare riferimento all'articolo di Birdman per individuare tramite [gdb e il codice sorgente PHP](https://www.laruence.com/2011/12/06/2381.html), il cui riassunto approssimativo dei passaggi è il seguente:
1. Eseguire il comando ```php -v``` per vedere la versione
2. [Scaricare il codice sorgente PHP corrispondente alla versione](https://www.php.net/releases/)
3. Eseguire il comando ```gdb --pid=pid del processo busy```
4. Eseguire il comando ```source percorso del codice sorgente PHP/.gdbinit```
5. Eseguire il comando ```zbacktrace``` per stampare lo stack di chiamate
Con l'ultimo passaggio si può vedere la posizione del loop nel codice PHP attualmente in esecuzione.
Nota: Se ```zbacktrace``` non stampa lo stack di chiamate, potrebbe essere necessario ricompilare PHP con il flag ```-g```, quindi riavviare workerman per individuare il problema.

**3. Aggiunta illimitata di timer**
Sezione di codice aggiunge costantemente dei timer senza rimuoverli, causando un accumulo illimitato di timer all'interno del processo, portando infine al funzionamento infinito del processo. Ad esempio:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    Timer::add(10, function(){});
};
Worker::runAll();
```
In questo esempio, ogni volta che un client si connette, viene aggiunto un timer, ma non c'è alcuna logica nel codice per rimuovere i timer. Con il passare del tempo, il numero di timer all'interno del processo aumenterà costantemente, portando infine all'esecuzione infinita dei timer e al processo in stato ```busy```. Il codice corretto dovrebbe essere:
```php
$worker = new Worker;
$worker->onConnect = function($con){
    $con->timer_id = Timer::add(10, function(){});
};
$worker->onClose = function($con){
    Timer::del($con->timer_id);
};
Worker::runAll();
```
