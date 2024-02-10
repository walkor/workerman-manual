# Anfragen sind in bestimmten Prozessen konzentriert

### Phänomen
Manchmal sehen wir beim Befehl `php start.php status`, dass Anfragen in bestimmten Prozessen verarbeitet werden, während andere Prozesse vollständig untätig sind.

### Preemption-Mechanismus
Workerman verwendet standardmäßig einen **präemptiven** Ansatz, um Verbindungen in mehreren Prozessen zu erhalten. Das bedeutet, dass wenn ein Client eine Verbindung herstellt, alle freien Prozesse die Möglichkeit haben, diese Verbindung zu erhalten, und derjenige, der am schnellsten ist, gewinnt. Wer am schnellsten ist, wird vom Betriebssystem-Kernel entschieden. Das Betriebssystem wählt möglicherweise bevorzugt den zuletzt verwendeten Prozess, um die CPU-Nutzung zu erhalten, da sich möglicherweise noch Kontextinformationen des vorherigen Prozesses im CPU-Register befinden, was den Kontextwechselaufwand reduzieren kann. Daher ist es wahrscheinlicher, dass während des Geschäftsablaufs oder während der Performancetests die Verbindungen von bestimmten Prozessen behandelt werden, da diese Strategie häufige Prozesswechsel vermeiden kann und die Leistung in der Regel optimal ist, und kein Problem darstellt.

### Rundruf-Mechanismus
Durch Setzen von `$worker->reusePort = true;` kann Workerman den Mechanismus zur Verbindungsaufnahme auf einen **Rundruf** umstellen. Im Rundruf-Modus verteilt der Kernel die Verbindungen gleichmäßig auf alle Prozesse, so dass alle Prozesse die Anfragen gemeinsam verarbeiten.

### Missverständnis
Viele Entwickler glauben, dass eine bessere Leistung entsteht, wenn alle Prozesse an der Verarbeitung von Anfragen beteiligt sind. Dies ist jedoch nicht unbedingt der Fall. Bei einfachen Geschäftsabläufen steigt die Leistung mit der Anzahl der an der Anfragenverarbeitung beteiligten Prozesse, die sich der Anzahl der CPU-Kerne des Servers annähert. Beispielsweise ist bei einem 4-Kern-Server in der Regel die höchste QPS bei 4 eingestellten Prozessen für einen Helloworld-Performancetest zu erwarten. Wenn die Anzahl der an der Verarbeitung beteiligten Prozesse die Anzahl der CPU-Kerne übersteigt, steigt der Kontextwechselaufwand, und die Leistung verschlechtert sich. Wenn jedoch eine Datenbank im Spiel ist, ist es möglicherweise besser, die Anzahl der Prozesse auf das 3- bis 6-Fache der CPU-Kerne zu erhöhen.
