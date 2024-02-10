# Disabilita il controllo delle funzioni

Utilizza questo script per verificare se ci sono funzioni disabilitate. Esegui il comando da riga di comando ```curl -Ss https://www.workerman.net/check | php```.

Se ricevi il messaggio ```La funzione nome_funzione potrebbe essere disabilitata. Si prega di controllare disable_functions in php.ini```, significa che le funzioni dipendenti da workerman sono disabiliate e devono essere abilitate nel file php.ini per poter utilizzare workerman correttamente.
Per abilitare le funzioni disabiliate, segui una tra le due opzioni indicate di seguito.

## Opzione uno: Script di correzione

Esegui lo script `curl -Ss https://www.workerman.net/fix | php` per abilitare le funzioni disabilitate.

## Opzione due: Abilitazione manuale

**Procedura:**

1. Esegui `php --ini` per individuare la posizione del file php.ini utilizzato da php cli.

2. Apri php.ini, individua la voce `disable_functions` e rimuovi la disabilitazione delle funzioni corrispondenti.

**Funzioni dipendenti**
Per utilizzare workerman, è necessario abilitare le seguenti funzioni:
``` 
stream_socket_server
stream_socket_client
pcntl_signal_dispatch
pcntl_signal
pcntl_alarm
pcntl_fork
posix_getuid
posix_getpwuid
posix_kill
posix_setsid
posix_getpid
posix_getpwnam
posix_getgrnam
posix_getgid
posix_setgid
posix_initgroups
posix_setuid
posix_isatty
```
