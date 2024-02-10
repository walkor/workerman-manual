# WorkerMan unterstützt eine hohe Anzahl von Nebenläufigkeiten

Der Begriff **Nebenläufigkeiten** ist zu vage, deshalb werden hier zwei quantifizierbare Indikatoren, nämlich die **Nebenläufige Verbindungszahl** und die **Nebenläufige Anfragenzahl**, verwendet, um dies zu erläutern.

Die **Nebenläufige Verbindungszahl** bezieht sich auf die Anzahl der aktiven TCP-Verbindungen, unabhängig davon, ob auf diesen Verbindungen Daten übertragen werden. Zum Beispiel kann ein Nachrichten-Push-Server möglicherweise Millionen von Geräteverbindungen aufrechterhalten, aber auf diesen Verbindungen findet nur selten Datenkommunikation statt. Daher kann die Auslastung dieses Servers nahezu bei 0 liegen, solange der verfügbare Speicher ausreicht, um weitere Verbindungen zuzulassen.

Die **Nebenläufige Anfragenzahl** wird in der Regel in QPS (Anfragen pro Sekunde) gemessen und kümmert sich weniger um die Anzahl der TCP-Verbindungen zu einem bestimmten Zeitpunkt auf dem Server. Zum Beispiel kann ein Server nur 10 Client-Verbindungen haben, aber jeder Client sendet 10.000 Anfragen pro Sekunde. In diesem Fall muss der Server mindestens eine Durchsatzleistung von 10 * 10.000 = 100.000 Anfragen pro Sekunde (QPS) bewältigen können. Angenommen, 100.000 Anfragen pro Sekunde stellen das Limit dieses Servers dar, dann könnte dieser Server 10 * 100.000 = 1.000.000 Clients unterstützen, wenn jeder Client pro Sekunde eine Anfrage an den Server sendet.

Die **Nebenläufige Verbindungszahl** ist durch den verfügbaren Arbeitsspeicher des Servers begrenzt. Im Allgemeinen kann ein Workerman-Server mit 24 GB Arbeitsspeicher ungefähr **1,2 Millionen** gleichzeitiger Verbindungen unterstützen.

Die **Nebenläufige Anfragenzahl** ist durch die CPU-Verarbeitungskapazität des Servers begrenzt. Ein 24-Kern-Workerman-Server kann eine Durchsatzleistung von etwa **450.000** Anfragen pro Sekunde (QPS) erreichen. Der tatsächliche Wert variiert je nach Geschäftskomplexität und Codequalität.

## Hinweis

Für Hochlastszenarien muss die event-Erweiterung installiert sein. Siehe Installations- und Konfigurationsabschnitt. Außerdem ist eine Optimierung des Linux-Kernels erforderlich, insbesondere in Bezug auf die Beschränkung der Anzahl der geöffneten Dateien pro Prozess. Siehe Anhang zum Thema Kernel-Optimierung.

## Testergebnisse

> Die Daten stammen von der renommierten Drittanbieter-Testorganisation techempower.com, Runde 20 des Leistungstests.
https://www.techempower.com/benchmarks/#section=data-r20&hw=ph&test=db&l=zik073-sf

**Serverkonfiguration:**
Insgesamt 14 Kerne, 28 Threads, 32 GB Arbeitsspeicher, dedizierter Cisco 10-Gigabit-Ethernet-Switch.
**Geschäftslogik:**
Mit Datenbankabfragen, PostgreSQL-Datenbank, PHP8 + JIT
QPS über 390.000+
![](../images/screenshot_1636522357217.png)
