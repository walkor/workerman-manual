# Channel verteiltes Kommunikationskomponente
**```(Erfordert Workerman-Version >= 3.3.0)```**

Quellcode-URL: https://github.com/walkor/Channel

Channel ist eine verteilte Kommunikationskomponente, die zur Kommunikation zwischen Prozessen oder Servern verwendet wird.

## Merkmale
1. Basierend auf dem Publish/Subscribe-Modell
2. Nicht blockierende E/A (Input/Output)

## Prinzip
Channel umfasst den Channel/Server-Server und den Channel/Client-Client.

Der Channel/Client verbindet sich über die connect-Schnittstelle mit dem Channel/Server und hält die Verbindung aufrecht.

Der Channel/Client informiert den Channel/Server über die on-Schnittstelle, welche Ereignisse er verfolgen soll, und registriert Ereignisrückruffunktionen (Rückrufe erfolgen im Prozess des Channel/Client).

Der Channel/Client veröffentlicht über die publish-Schnittstelle ein bestimmtes Ereignis und die damit verbundenen Daten an den Channel/Server.

Der Channel/Server empfängt das Ereignis und die Daten und verteilt sie an den Channel/Client, der dieses Ereignis verfolgt.

Nachdem der Channel/Client das Ereignis und die Daten erhalten hat, wird der gemäß der on-Schnittstelle festgelegte Rückruf ausgelöst.

Der Channel/Client empfängt nur die Ereignisse, die er verfolgt hat, und löst die Rückrufe aus.

## Installation
`composer require workerman/channel`

## Beachtung
Channel kann nur in einer Workerman-Umgebung verwendet werden. Es kann nicht in einer PHP-FPM-Umgebung verwendet werden.
