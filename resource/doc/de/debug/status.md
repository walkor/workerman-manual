# Überprüfen des Ausführungsstatus

Durch Ausführen von `php start.php status` können Sie den Ausführungsstatus von Workerman überprüfen, ähnlich wie folgt:

```  
----------------------------------------------GLOBALER STATUS----------------------------------------------------
Workerman-Version: 3.5.13          PHP-Version: 5.5.9-1ubuntu4.24
Startzeit: 2018-02-03 11:48:20   Laufzeit 112 Tage 2 Stunden   
Durchschnittliche Auslastung: 0, 0, 0            Ereignisschleife: \Workerman\Events\Event
4 Worker       11 Prozesse
worker_name        exit_status      exit_count
ChatBusinessWorker 0                0
ChatGateway        0                0
Register           0                0
WebServer          0                0
----------------------------------------------PROZESSSTATUS---------------------------------------------------
pid	Speicher  Listening                worker_name        Verbindungen send_fail Timer  Gesamtanfragen QPS    Status
18306	2,25M   keine                     ChatBusinessWorker 5           0         0       11            0      [inaktiv]
18307	2,25M   keine                     ChatBusinessWorker 5           0         0       8             0      [inaktiv]
18308	2,25M   keine                     ChatBusinessWorker 5           0         0       3             0      [inaktiv]
18309	2,25M   keine                     ChatBusinessWorker 5           0         0       14            0      [inaktiv]
18310	2M      websocket://0.0.0.0:7272 ChatGateway        8           0         1       31            0      [inaktiv]
18311	2M      websocket://0.0.0.0:7272 ChatGateway        7           0         1       26            0      [inaktiv]
18312	2M      websocket://0.0.0.0:7272 ChatGateway        6           0         1       21            0      [inaktiv]
18313	1,75M   websocket://0.0.0.0:7272 ChatGateway        5           0         1       16            0      [inaktiv]
18314	1,75M   text://0.0.0.0:1236      Register           8           0         0       8             0      [inaktiv]
18315	1,5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [inaktiv]
18316	1,5M    http://0.0.0.0:55151     WebServer          0           0         0       0             0      [inaktiv]
----------------------------------------------PROZESSSTATUS---------------------------------------------------
Zusammenfassung: 18M     -                        -                  54          0         4       138           0      [Zusammenfassung]
```

## Erklärung

### GLOBALER STATUS

Aus dieser Spalte können Sie folgende Informationen ablesen:

- Workerman-Version ```version:3.5.13```
- Startzeit ```2018-02-03 11:48:20```, Laufzeit ```run 112 Tage 2 Stunden```
- Serverauslastung ```load average: 0, 0, 0```, die Durchschnittsbelastung des Systems in den letzten 1, 5 und 15 Minuten
- Verwendete IO-Ereignisbibliothek, ```event-loop:\Workerman\Events\Event```
- ```4 Worker``` (3 Arten von Prozessen, einschließlich ChatGateway, ChatBusinessWorker, Register-Prozesse, WebServer-Prozess)
- ```11 Prozesse``` (insgesamt 11 Prozesse)
- ```worker_name``` (Name des Worker-Prozesses)
- ```exit_status``` (Ausgangsstatus des Worker-Prozesses)
- ```exit_count``` (Anzahl der Ausgänge mit diesem Status)

Im Allgemeinen zeigt ein `exit_status` von 0 einen normalen Ausstieg an. Wenn der Wert jedoch anders ist, bedeutet dies, dass der Prozess unerwartet beendet wurde, und es wird eine Fehlermeldung wie "WORKER EXIT UNEXPECTED" generiert, die in der Datei [Worker::logFile](worker/log-file.md) protokolliert wird.

**Gängige `exit_status` und ihre Bedeutung sind:**

- 0: Normaler Ausstieg, tritt auf, wenn ein Reload-Prozess zum sanften Neustart ausgeführt wird. Hinweis: Das Aufrufen von exit oder die Anweisung die in der Anwendung zu einem `exit_status` von 0 führen, führt zu einer "WORKER EXIT UNEXPECTED" Fehlermeldung. Workerman erlaubt es der Geschäftslogik nicht, die exit- oder die die Anweisung zu verwenden.
- 9: Der Prozess wurde durch das SIGKILL-Signal getötet. Dieser `exit_status` tritt hauptsächlich bei stop und Reload-Prozessen auf, weil die Kindprozesse nicht innerhalb der vorgegebenen Zeit auf das Reload-Signal des Hauptprozesses reagieren (z.B. lang andauernde Blockaden durch mysql, curl oder Geschäfts-Deadlocks), und somit gezwungen werden, mit dem SIGKILL-Signal beendet zu werden. Es ist zu beachten, dass das Senden des SIGKILL-Signals an den Kindprozess mit dem kill Befehl in der Linux-Befehlszeile ebenfalls zu diesem `exit_status` führen kann.
- 11: Ein PHP-Core-Dump ist aufgetreten, was hauptsächlich auf die Verwendung instabiler Erweiterungen zurückzuführen ist. In diesem Fall sollten Sie die entsprechenden Erweiterungen in der php.ini auskommentieren. In einigen Fällen kann es auch an einem PHP-Bug liegen, in diesem Fall ist ein Upgrade von PHP erforderlich
- 65280: Ein `exit_status` von 65280 zeigt an, dass ein schwerwiegender Fehler in der Geschäftslogik aufgetreten ist, wie beispielsweise der Aufruf einer nicht vorhandenen Funktion oder ein Syntaxfehler. Die spezifische Fehlerinformation wird in der Datei [Worker::logFile](worker/log-file.md) protokolliert und kann auch in der Datei, die durch die `error_log`-Einstellung in der [php.ini](https://php.net/manual/zh/ini.list.php) festgelegt ist (falls vorhanden), gefunden werden.
- 64000: Dieser `exit_status` tritt auf, wenn die Geschäftslogik eine Ausnahme wirft, die von der Geschäftslogik nicht abgefangen wurde und zum Abbruch des Prozesses führt. Wenn Workerman im Debug-Modus ausgeführt wird, wird der Ausnahmestapel auf dem Terminal ausgegeben; im Daemon-Modus wird der Ausnahmestapel in der Datei [Worker::stdoutFile](worker/stdout-file.md) festgelegt.

## PROZESSSTATUS

pid: Prozess-PID

Speicher: Der derzeitige Speicherverbrauch dieses Prozesses (ohne den Speicherbedarf der ausführbaren PHP-Datei selbst)

Listening: Transportprotokoll und Überwachung von IP und Port. Wenn kein Port überwacht wird, wird "keine" angezeigt. Siehe [Worker-Konstruktor](worker/construct.md).

worker_name: Der Name des Dienstes, der in diesem Prozess ausgeführt wird. Siehe [Worker-Namenseigenschaft](worker/name.md).

Verbindungen: Die aktuelle Anzahl der TCP-Verbindungsinstanzen in diesem Prozess, einschließlich der TcpConnection- und AsyncTcpConnection-Instanzen. Dieser Wert ist ein Echtzeitwert und kein kumulativer Wert. Hinweis: Wenn nach dem Schließen der Verbindungsinstanz die entsprechende Anzahl nicht abnimmt, könnte dies daran liegen, dass der Geschäftscode das $connection-Objekt speichert und daher die Verbindungsinstanz nicht zerstört wird.

total_request: Die Gesamtzahl der Anfragen, die dieser Prozess seit dem Start erhalten hat. Diese Anzahl umfasst nicht nur Anfragen von Clients, sondern auch interne Workerman-Kommunikationsanfragen, wie z.B. zwischen Gateway und Business Worker in der GatewayWorker-Architektur. Dieser Wert ist ein kumulativer Wert.

send_fail: Die Anzahl der fehlgeschlagenen Versuche dieses Prozesses, Daten an Clients zu senden. Ein Nicht-Null-Wert gilt in der Regel als normaler Zustand und wird in den [status-Informationen](../faq/about-send-fail.md) erläutert. Dieser Wert ist ein kumulativer Wert.

Timers: Die Anzahl der aktiven Timer dieses Prozesses (ohne gelöschte Timer und einmalige Timer, die bereits ausgeführt wurden). Hinweis: Diese Funktion erfordert Workerman Version >= 3.4.7. Dieser Wert ist ein Echtzeitwert und kein kumulativer Wert.

qps: Die Anzahl der Netzwerkanfragen, die dieser Prozess pro Sekunde empfängt. Hinweis: Nur bei der Verwendung von `status -d` wird dieser Wert statistisch erfasst, ansonsten wird 0 angezeigt. Diese Funktion erfordert Workerman Version >= 3.5.2. Dieser Wert ist ein Echtzeitwert und kein kumulativer Wert.

status: Der Status des Prozesses, "inaktiv" bedeutet, dass der Prozess inaktiv ist und "beschäftigt", dass der Prozess gerade beschäftigt ist. Hinweis: Wenn ein Prozess vorübergehend beschäftigt ist, ist dies normal, aber wenn ein Prozess dauerhaft beschäftigt ist, besteht die Möglichkeit, dass eine Geschäftsblockierung oder eine Geschäftsschleife auftritt, die anhand des Abschnitts [Troubleshooting Busy Processes](busy-process.md) überprüft werden sollte. Hinweis: Diese Funktion erfordert Workerman Version >= 3.5.0.

## Funktionsweise
Nach dem Starten des status-Skripts sendet der Hauptprozess ein `SIGUSR2`-Signal an alle Worker-Prozesse. Anschließend wechselt das status-Skript in eine kurze Schlafphase, um auf die statistischen Ergebnisse der einzelnen Worker-Prozesse zu warten. Während dieser Zeit schreiben die inaktiven Worker-Prozesse sofort ihren Betriebsstatus (Anzahl der Verbindungen, Anzahl der Anfragen usw.) in eine spezielle Festplattendatei, während die Worker-Prozesse, die gerade die Geschäftslogik verarbeiten, auf das Abschließen der Geschäftslogik warten, bevor sie ihren Betriebsstatus schreiben. Nach der kurzen Schlafphase beginnt das status-Skript, die Betriebsstatusdateien von der Festplatte zu lesen und die Ergebnisse in der Konsole anzuzeigen.

## Beachten
Beim status-Befehl können in einigen Fällen Prozesse als "beschäftigt" angezeigt werden, wenn der Prozess mit der Verarbeitung der Geschäftsvorgänge beschäftigt ist (z. B. lange Blockaden bei curl- oder Datenbankanfragen oder lange Schleifen), wodurch der Status nicht aktualisiert werden kann und daher als "beschäftigt" angezeigt wird.

In diesem Fall gilt es, den Geschäftscode zu überprüfen, um festzustellen, wo die Geschäftsblockade auftritt, und die Blockadezeit zu bewerten. Wenn die Blockadezeit nicht den Erwartungen entspricht, ist es notwendig, den Geschäftscode gemäß dem Abschnitt [Troubleshooting Busy Processes](busy-process.md) zu überprüfen.
