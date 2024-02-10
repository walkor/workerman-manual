# Avvertenze
## Avvertenze sull'uso del timer

1. I timer possono essere aggiunti solo nei callback ```onXXXX```. Si consiglia di impostare i timer globali nel callback ```onWorkerStart``` e i timer specifici per una connessione nel callback ```onConnect```.

2. I compiti programmati vengono eseguiti nel processo corrente (non vengono avviati nuovi processi o thread). Se il compito è molto pesante (soprattutto se coinvolge operazioni di I/O di rete), potrebbe causare il blocco del processo e l'incapacità temporanea di gestire altri task. Pertanto, è meglio eseguire i compiti pesanti in un processo separato, ad esempio avviando uno o più processi Worker dedicati.

3. Se il processo è occupato con altre attività o se un compito non viene completato entro il tempo previsto, e nel frattempo è arrivato il momento di eseguire un nuovo ciclo del timer, il sistema aspetterà che il compito corrente venga completato prima di eseguire il nuovo ciclo. Ciò può causare l'incorrere in un intervallo di esecuzione del timer diverso da quello previsto. In altre parole, le attività del processo corrente vengono eseguite in modo seriale, mentre in presenza di più processi, le attività vengono eseguite in parallelo.

4. Bisogna prestare attenzione quando si impostano compiti programmati per più processi, ad esempio il seguente codice stamperà "hi" cinque volte al secondo:
   ```php
   $worker = new Worker();
   // 5 processi
   $worker->count = 5;
   $worker->onWorkerStart = function(Worker $worker) {
       // 5 processi, ognuno con un timer simile
       Timer::add(1, function(){
           echo "hi\r\n";
       });
   };
   Worker::runAll();
   ```
   Se si vuole che solo un processo esegua il timer, consultare l'esempio 2 in [Timer::add](add.md).

5. Potrebbe verificarsi un margine di errore di circa 1 millisecondo.

6. I timer non possono essere eliminati trasversalmente tra i processi. Ad esempio, un timer impostato dal processo A non può essere eliminato direttamente dal processo B chiamando l'interfaccia Timer::del.

7. Gli ID dei timer tra processi diversi potrebbero essere duplicati, ma all'interno dello stesso processo gli ID dei timer non saranno duplicati.

8. Cambiare l'ora di sistema influenzerà il comportamento dei timer, pertanto si consiglia di riavviare con restart dopo aver modificato l'ora di sistema.
