# Funzioni non supportate

Funzione / Istruzione non supportata | Alternativa | Spiegazione
----|------|----
pcntl_fork | Imposta in anticipo il numero di processi | 
php://input | [`$request->rawBody()`](http/request.md)| Utilizzato dalle applicazioni nell'ambito del protocollo HTTP per ottenere i dati POST grezzi
exit | return | L'uso di exit provoca l'uscita del processo, se si desidera restituire un valore, utilizzare direttamente l'istruzione return
die | return | L'uso di die provoca l'uscita del processo, se si desidera restituire un valore, utilizzare direttamente l'istruzione return
Funzioni relative a header, cookie e session | Fare riferimento alle classi [`$request`](http/request.md) e [`$response`](http/response.md) | 
set_time_limit| N/D | Può essere impostato solo su 0, in caso contrario farà terminare il processo workerman dopo un certo periodo di tempo
