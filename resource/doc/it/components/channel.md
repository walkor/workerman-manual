# Componente di comunicazione distribuita Channel
**``` (Richiede Workerman versione >= 3.3.0) ```**

Repository del codice sorgente: https://github.com/walkor/Channel

Channel è un componente di comunicazione distribuita utilizzato per la comunicazione tra processi o server.

## Caratteristiche
1. Basato sul modello di pubblicazione/sottoscrizione
2. IO non bloccante

## Principio
Channel include il server Channel/Server e il client Channel/Client.
Il client Channel/Client si connette al server Channel/Server tramite l'interfaccia connect e mantiene una connessione a lungo termine.
Il client Channel/Client utilizza l'interfaccia on per informare il server Channel/Server su quali eventi sta seguendo e registra le funzioni di callback degli eventi (il callback avviene nel processo in cui si trova il client Channel/Client).
Il client Channel/Client pubblica un evento e i dati correlati al server Channel/Server tramite l'interfaccia publish.
Dopo aver ricevuto l'evento e i dati, il server Channel/Server li distribuisce ai client Channel/Client interessati a tale evento.
Il client Channel/Client riceve l'evento e i dati e attiva i callback impostati tramite l'interfaccia on.
Il client Channel/Client riceverà solo gli eventi a cui è interessato e attiverà i callback.

## Installazione
`composer require workerman/channel`

## Nota
Channel può essere utilizzato solo in un ambiente Workerman e non può essere utilizzato in un ambiente php-fpm.
