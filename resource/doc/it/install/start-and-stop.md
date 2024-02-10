# Avvio e arresto

Si noti che i comandi di avvio e arresto di Workerman vengono eseguiti nella riga di comando.

Per avviare Workerman, è prima necessario avere un file di ingresso di avvio che definisca la porta e il protocollo di ascolto del servizio. Puoi fare riferimento alla sezione [Esempio di sviluppo semplice](../getting-started/simple-example.md) per ottenere ulteriori informazioni.

Prendiamo ad esempio [workerman-chat](https://www.workerman.net/workerman-chat), il suo file di avvio è start.php.

### Avvio

Avviare in modalità debug:

```php start.php start```

Avviare in modalità demone:

```php start.php start -d```

### Arresto

```php start.php stop```

### Riavvio

```php start.php restart```

### Riavvio fluido

```php start.php reload```

### Stato

```php start.php status```

### Stato della connessione (richiede Workerman versione >=3.5.0)

```php start.php connections```

## Differenze tra modalità debug e demone

1. Avviando in modalità debug, le funzioni di output come echo, var_dump, print vengono stampate direttamente nel terminale.

2. Avviando in modalità demone, le funzioni di output come echo, var_dump, print verranno reindirizzate per impostazione predefinita al file /dev/null. È possibile impostare il percorso del file con `Worker::$stdoutFile = '/tuo/percorso/file';`.

3. Avviando in modalità debug, Workerman si chiuderà e uscirà quando si chiude il terminale.

4. Avviando in modalità demone, Workerman continuerà a funzionare normalmente in background dopo la chiusura del terminale.

## Cos'è il riavvio fluido?

Vedi [Principi di riavvio fluido](../faq/reload-principle.md)
