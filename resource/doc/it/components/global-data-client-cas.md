# cas
**``` (Richiede Workerman versione>=3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
Sostituzione atomica, sostituisci ```$old_value``` con ```$new_value```.
Solo quando il valore corrispondente a questa chiave non è stato modificato da altri client dopo l'ultima lettura da parte del client corrente, è possibile scrivere il valore.

## Parametri

 ``` $key ```

Chiave. (Ad esempio ```$global->abc```, ```abc``` è la chiave)

 ``` $old_value ```

Dati vecchi

 ``` $new_value ```

Dati nuovi

## Valore restituito
Restituisce true se la sostituzione ha avuto successo, altrimenti restituisce false.

## Spiegazione:

Quando più processi operano contemporaneamente sulla stessa variabile condivisa, a volte è necessario considerare il problema della concorrenza.

Ad esempio, due processi A e B aggiungono contemporaneamente un membro all'elenco degli utenti.
Attualmente l'elenco degli utenti per i processi A e B è ```$global->user_list = array(1,2,3)```.
Il processo A opera sulla variabile ```$global->user_list```, aggiungendo un utente 4.
Il processo B opera sulla variabile ```$global->user_list```, aggiungendo un utente 5.
Il processo A imposta la variabile ```$global->user_list = array(1,2,3,4)``` con successo.
Il processo B imposta la variabile ```$global->user_list = array(1,2,3,5)``` con successo.
In questo momento, la variabile impostata dal processo B sovrascrive la variabile impostata dal processo A, causando la perdita dei dati.

Ciò è dovuto al fatto che la lettura e la scrittura non sono un'operazione atomica, causando problemi di concorrenza.
Per risolvere questo problema di concorrenza, è possibile utilizzare l'interfaccia di sostituzione atomica cas.
Prima di modificare un valore, l'interfaccia cas verifica se questo valore è stato modificato da altri processi in base a ```$old_value```.
Se è stato modificato, non viene sostituito e restituisce false. Altrimenti restituisce true.
Vedere l'esempio seguente.

 **Nota:** 
Alcuni dati condivisi sovrascritti da concorrenza non sono un problema, ad esempio il sistema di asta attuale offerta massima o l'attuale inventario di un certo prodotto.

## Esempio

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Inizializzazione della lista
$global->user_list = array(1,2,3);

// Aggiungi in modo atomico un valore a user_list
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
