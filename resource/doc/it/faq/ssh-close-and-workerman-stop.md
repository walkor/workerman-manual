# Chiusura del terminale che causa la chiusura del servizio
**Domanda:**

Perché quando chiudo il terminale, Workerman si chiude da solo?

**Risposta:**

Workerman ha due modalità di avvio, modalità di debug e modalità daemon. 

Eseguire ```php xxx.php start``` avvia la modalità di debug, utilizzata per lo sviluppo e il debug dei problemi; quando si chiude il terminale, Workerman si chiude insieme.

Eseguire ```php xxx.php start -d``` avvia la modalità daemon, in cui la chiusura del terminale non influisce su Workerman.

Se si desidera che Workerman non sia influenzato dal terminale, è possibile avviarlo in modalità daemon.
