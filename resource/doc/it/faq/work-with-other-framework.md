# Come integrare con altri framework

**Domanda:**

Come posso integrare workerman con altri framework MVC (come thinkPHP, Yii, ecc.)?

**Risposta:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

È **consigliato** integrare workerman con altri framework MVC come mostrato nell'immagine sopra (usando thinkPHP come esempio):

1. ThinkPHP e Workerman sono due sistemi indipendenti e possono essere distribuiti separatamente (anche su server diversi) senza interferire l'uno con l'altro.

2. ThinkPHP fornisce pagine web renderizzate nei browser utilizzando il protocollo HTTP.

3. Le pagine fornite da ThinkPHP possono stabilire una connessione websocket con Workerman utilizzando JavaScript.

4. Dopo la connessione, viene inviato un pacchetto di dati a Workerman (contenente nome utente, password o qualche token) per verificare a quale utente appartiene la connessione websocket.

5. Workerman viene chiamato solo quando ThinkPHP ha bisogno di inviare dati al browser tramite l'interfaccia socket.

6. Le altre richieste vengono comunque gestite nel modo normale dal framework ThinkPHP tramite HTTP.

**In sintesi:**

Workerman viene utilizzato come un canale per l'invio di dati al browser e viene chiamato solo quando è necessario inviare dati al browser. Tutta la logica di business viene gestita da ThinkPHP.

Per informazioni su come ThinkPHP chiama l'interfaccia socket di Workerman per inviare dati, fare riferimento alla sezione "Domande frequenti - Push in altri progetti" nel documento [Qui](push-in-other-project.md).

**ThinkPHP supporta ufficialmente workerman, consulta il [manuale di ThinkPHP5](https://www.kancloud.cn/manual/thinkphp5/235128)**
