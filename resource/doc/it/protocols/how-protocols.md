## Come personalizzare un protocollo

In realtà, è abbastanza semplice creare il proprio protocollo. Un protocollo semplice di solito consiste in due parti:
 * Identificatore per distinguere i limiti dei dati
 * Definizione del formato dei dati

## Un esempio

### Definizione del protocollo
Supponiamo che l'identificatore per distinguere i limiti dei dati sia il carattere di nuova riga "\n" (nota: i dati stessi non devono contenere il carattere di nuova riga) e che il formato dei dati sia JSON. Di seguito è riportato un esempio di un pacchetto di richiesta che rispetta questa regola.

<pre>
{"type":"message","content":"ciao"}
</pre>

Si noti che alla fine dei dati della richiesta c'è un carattere di nuova riga (rappresentato in PHP dalla stringa **"\n"** tra **doppie virgolette**), che indica la fine di una richiesta.

### Passaggi per l'implementazione
Per implementare il protocollo sopra descritto in Workerman, supponiamo che il protocollo sia chiamato JsonNL e che il progetto si chiami MyApp. Quindi è necessario seguire i seguenti passaggi:

1. Posizionare il file del protocollo nella cartella Protocols del progetto, ad esempio il file MyApp/Protocols/JsonNL.php.

2. Implementare la classe JsonNL, utilizzare ```namespace Protocols;``` come spazio dei nomi e assicurarsi di implementare tre metodi statici: input, encode e decode.

Nota: Workerman chiamerà automaticamente questi tre metodi statici per implementare la divisione, l'encoding e il decoding dei pacchetti. Per i dettagli sul processo, fare riferimento alle istruzioni sul flusso d'esecuzione più avanti.

### Flusso di interazione tra Workerman e la classe di protocollo
1. Supponiamo che il client invii un pacchetto di dati al server. Il server, una volta ricevuti i dati (anche se parziali), invoca immediatamente il metodo ```input`` del protocollo per determinare la lunghezza di questo pacchetto. Il metodo ```input``` restituisce il valore di lunghezza ```$length``` a Workerman.
2. Una volta che Workerman ha ricevuto il valore ```$length```, controlla se c'è abbastanza dati nel buffer per raggiungere la lunghezza ```$length```. Se non ci sono abbastanza dati, Workerman continua ad attendere fino a quando la lunghezza dei dati nel buffer non sarà almeno pari a ```$length```.
3. Una volta che la lunghezza dei dati nel buffer è sufficiente, Workerman estrae i dati di lunghezza ```$length``` dal buffer (ovvero **divisione dei pacchetti**) e chiama il metodo ```decode``` del protocollo per **decodificare** i dati, ottenendo ```$data```.
4. Dopo la decodifica, Workerman trasmette i dati ```$data``` al business mediante il callback ```onMessage($connection, $data)```. All'interno di ```onMessage``` il business può utilizzare la variabile ```$data``` per ottenere i dati completi e decodificati inviati dal client.
5. Quando il business deve inviare dati al client chiamando il metodo ```$connection->send($buffer)```, Workerman utilizzerà automaticamente il metodo ```encode``` del protocollo per **codificare** ```$buffer``` prima di inviarlo al client.

### Implementazione dettagliata

**Implementazione di MyApp/Protocols/JsonNL.php**

```php
namespace Protocols;
class JsonNL
{
    /**
     * Controlla l'integrità del pacchetto
     * Se è possibile ottenere la lunghezza del pacchetto in $buffer, restituisce la lunghezza dell'intero pacchetto.
     * Altrimenti restituisce 0 per continuare ad attendere altri dati.
     * Se restituisce false o un valore negativo, viene interpretato come richiesta errata e la connessione viene interrotta.
     * @param string $buffer
     * @return int
     */
    public static function input($buffer)
    {
        // Trova la posizione del carattere di nuova riga "\n"
        $pos = strpos($buffer, "\n");
        // Se non c'è un carattere di nuova riga, la lunghezza del pacchetto non è nota, pertanto restituisce 0 per continuare ad attendere altri dati
        if($pos === false)
        {
            return 0;
        }
        // Se c'è un carattere di nuova riga, restituisce la lunghezza attuale del pacchetto (incluso il carattere di nuova riga)
        return $pos+1;
    }

    /**
     * Codifica: chiamato automaticamente quando si invia dati al client
     * @param string $buffer
     * @return string
     */
    public static function encode($buffer)
    {
        // Serializzazione in formato JSON, con aggiunta di un carattere di nuova riga come segnalazione di fine richiesta
        return json_encode($buffer)."\n";
    }

    /**
     * Decodifica: chiamato automaticamente quando il numero di byte dei dati ricevuti è pari al valore restituito da input (ovvero, un valore maggiore di 0)
     * Questi dati vengono poi passati automaticamente come parametro $data alla funzione di callback onMessage
     * @param string $buffer
     * @return string
     */
    public static function decode($buffer)
    {
        // Rimuove il carattere di nuova riga e ripristina il formato in array
        return json_decode(trim($buffer), true);
    }
}
```

Con questo, l'implementazione del protocollo JsonNL è completata e può essere utilizzata all'interno del progetto MyApp, come mostrato di seguito.

File: MyApp\start.php

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$json_worker = new Worker('JsonNL://0.0.0.0:1234');
$json_worker->onMessage = function(TcpConnection $connection, $data) {

    // $data contiene i dati inviati dal client e già decodificati utilizzando JsonNL::decode
    echo $data;
    
    // I dati inviati tramite $connection->send verranno automaticamente codificati utilizzando il metodo JsonNL::encode prima di essere inviati al client
    $connection->send(array('code'=>0, 'msg'=>'ok'));
    
};
Worker::runAll();
...
```

> **Nota**
> Workerman cercherà di caricare il protocollo in namespace `Protocols`. Ad esempio, `new Worker('JsonNL://0.0.0.0:1234')` cercherà di caricare il protocollo `Protocols\JsonNL`.
> Se si verifica un errore del tipo `Class 'Protocols\JsonNL' non trovata`, fare riferimento all'[implementazione del caricamento automatico](../faq/autoload.md).

### Spiegazione dell'interfaccia del protocollo
Le classi di protocollo sviluppate in Workerman devono implementare tre metodi statici: input, encode e decode. Per le specifiche dell'interfaccia del protocollo, fare riferimento a Workerman/Protocols/ProtocolInterface.php, come definito di seguito:

```php
namespace Workerman\Protocols;

use \Workerman\Connection\ConnectionInterface;

/**
 * Interfaccia del protocollo
* Autore: walkor <walkor@workerman.net>
 */
interface ProtocolInterface
{
    /**
     * Utilizzato per suddividere i dati in $recv_buffer
     *
     * Se è possibile ottenere la lunghezza del pacchetto in $recv_buffer, restituisce la lunghezza dell'intero pacchetto.
     * Altrimenti restituisce 0, indicando che sono necessari più dati per ottenere la lunghezza corrente del pacchetto.
     * Se restituisce false o un valore negativo, viene interpretato come richiesta errata e la connessione viene interrotta.
     *
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     * @return int|false
     */
    public static function input($recv_buffer, ConnectionInterface $connection);

    /**
     * Utilizzato per decodificare il pacchetto
     *
     * Se input restituisce un valore maggiore di 0 e Workerman ha ricevuto abbastanza dati, viene chiamato automaticamente decode.
     * Questo avvia anche la callback onMessage e passa i dati decodificati come secondo parametro al callback.
     * In altre parole, quando il client invia una richiesta completa, viene chiamato automaticamente decode per decodificare i dati, senza la necessità di una chiamata manuale all'interno del codice del business.
     * @param ConnectionInterface $connection
     * @param string $recv_buffer
     */
    public static function decode($recv_buffer, ConnectionInterface $connection);

    /**
     * Utilizzato per codificare il pacchetto
     *
     * Quando il business invia dati al client tramite $connection->send($data), i dati vengono automaticamente codificati utilizzando encode, creando un formato di dati conforme al protocollo, prima di essere inviati al client
     * In altre parole, i dati inviati al client vengono automaticamente codificati con encode, senza la necessità di una chiamata manuale nel codice del business.
     * @param ConnectionInterface $connection
     * @param mixed $data
     */
    public static function encode($data, ConnectionInterface $connection);
}
```

## Nota:
In Workerman non è strettamente richiesto che le classi di protocollo implementino l'interfaccia di ProtocolInterface. In realtà, è sufficiente che una classe di protocollo contenga i tre metodi statici input, encode e decode.
