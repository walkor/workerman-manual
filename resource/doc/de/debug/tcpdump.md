# Netzwerk-Paketmitschnitt

Im folgenden Beispiel verwenden wir ```tcpdump```, um die Daten zu überwachen, die durch die Anwendung ```workerman-chat``` über das ```Websocket```-Protokoll übertragen werden. In dem Beispiel von ```workerman-chat``` dient der Server zur Bereitstellung von ```Websocket```-Diensten über den Port ```7272```. Daher erfassen wir die Datenpakete auf dem Port ```7272```.

1. *Befehl ausführen:* ```tcpdump -Ans 4096 -iany port 7272```

2. Geben Sie in die Adressleiste des Browsers ein: ```http://127.0.0.1:55151```

3. Geben Sie einen Spitznamen ein: ```mynick```

4. Geben Sie in das Eingabefeld ein: ```Hi, all!```

*Die erfassten Daten sind wie folgt:*

```plaintext
/*
 * Erste TCP-Verbindungsherstellung
 * Die lokale Portnummer 60653 sendet SYN-Paket an den Remote-Port 7272
 */
17:50:00.523910 IP 127.0.0.1.60653 > 127.0.0.1.7272: Flags [S], seq 3524290970, win 32768, options [mss 16396,sackOK,TS val 28679666 ecr 28679554,nop,wscale 7], length 0
E..<.h@.@.HQ...........h..i..........0....@....
............

// Weitere Pakete und deren Beschreibungen folgen...
```

*Die Daten enthalten die Beschreibung der TCP- und Websocket-Handshakes sowie der Datentransfer zwischen Client und Server.*

*Hinweis: Die Angabe in eckigen Klammern, wie z.B. ```[S]```, kennzeichnet den Pakettyp (z.B. SYN für Verbindungsaufbau). Bei Bedarf können auch hexadezimale Darstellungen für die Daten mittels des Befehls ```tcpdump -XAns 4096 -iany port 7272``` angezeigt werden.*

*Alle oben genannten Anfragen und Nachrichten repräsentieren die Anmeldung und das Chat-Protokoll zwischen verschiedenen Browser-Clients.*
