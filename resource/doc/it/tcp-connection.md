# Interfacce fornite dalla classe Connection

In WorkerMan ci sono due classi importanti: Worker e Connection.

Ogni connessione client corrisponde a un oggetto Connection, che consente di impostare callback come onMessage, onClose, e fornisce anche un'interfaccia per inviare dati al client tramite il metodo send e per chiudere la connessione tramite il metodo close, insieme ad altre interfacce necessarie.

Si può dire che Worker è un contenitore di ascolto, responsabile per accettare le connessioni client e fornire agli sviluppatori l'oggetto connection per operare su di esso.
