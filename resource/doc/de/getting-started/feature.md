# WorkerMan Eigenschaften

### 1. Reine PHP Entwicklung
Anwendungen, die mit WorkerMan entwickelt wurden, sind unabhängig von Containern wie php-fpm, apache und nginx und können eigenständig ausgeführt werden. Dies ermöglicht es PHP-Entwicklern, Anwendungen einfach zu entwickeln, bereitzustellen und zu debuggen.

### 2. Unterstützung von PHP-Mehrprozessen
Um die Leistung mehrerer CPUs voll auszuschöpfen, unterstützt WorkerMan standardmäßig Mehrprozess-Multitasking. WorkerMan startet einen Hauptprozess und mehrere Unterprozesse, die Dienste nach außen bereitstellen. Der Hauptprozess überwacht die Unterprozesse, und die Unterprozesse überwachen die Netzwerkverbindungen und empfangen und verarbeiten Daten. Aufgrund des einfachen Prozessmodells wird WorkerMan stabiler und effizienter.

### 3. Unterstützung von TCP und UDP
WorkerMan unterstützt die beiden Transportprotokolle TCP und UDP, und es ist nur eine Konfigurationsänderung erforderlich, um das Transportprotokoll zu wechseln, ohne dass der Geschäftscode geändert werden muss.

### 4. Unterstützung von Langzeitverbindungen
Oft müssen PHP-Anwendungen eine Langzeitverbindung zum Client aufrechterhalten, z. B. für Chatrooms, Spiele usw., was jedoch in herkömmlichen PHP-Containern (apache, nginx, php-fpm) schwer zu realisieren ist. Mit WorkerMan kann eine PHP-Langzeitverbindung verwendet werden, solange der Serverdienst nicht aktiv die Verbindungsabbrechung aufruft. Ein einzelner WorkerMan-Prozess kann Tausende von gleichzeitigen Verbindungen unterstützen, während mehrere Prozesse Hunderttausende oder sogar Millionen von parallelen Verbindungen unterstützen.

### 5. Unterstützung verschiedener Anwendungsprotokolle
WorkerMan bietet Schnittstellenunterstützung für verschiedene Anwendungsprotokolle, einschließlich benutzerdefinierter Protokolle. Das Wechseln des Protokolls in WorkerMan ist ebenfalls sehr einfach und erfordert nur eine Konfiguration. Dadurch bleibt der Geschäftscode unverändert, und es können sogar mehrere Ports für unterschiedliche Clientanforderungen geöffnet werden.

### 6. Unterstützung hoher Parallelität
WorkerMan unterstützt die Libevent Event-Loop-Bibliothek (die event-Erweiterung muss installiert sein). Die Verwendung von Event bietet eine ausgezeichnete Leistung bei langfristigen, parallelen Verbindungen. Wenn die Event-Erweiterung nicht installiert ist, verwendet WorkerMan standardmäßig die integrierten Select-Systemaufrufe, die ebenfalls eine sehr starke Leistung bieten.

### 7. Unterstützung eines reibungslosen Dienstrestarts
Beim Neustart des Dienstes (z. B. bei der Veröffentlichung einer neuen Version) möchten wir nicht, dass die Prozesse, die gerade Benutzeranfragen verarbeiten, sofort beendet werden, und erst recht möchten wir nicht, dass die Kommunikation mit den Clients in diesem Moment fehlschlägt. WorkerMan bietet eine Funktion für einen reibungslosen Neustart, die einen reibungslosen Upgrade des Dienstes gewährleistet, ohne die Nutzung durch die Clients zu beeinträchtigen.

### 8. Unterstützung für die Überwachung und automatische Aktualisierung von Dateien
Während der Entwicklung möchten wir, dass Änderungen im Code sofort wirksam werden, um die Ergebnisse zu überprüfen. WorkerMan bietet eine [FileMonitor-Dateiüberwachungskomponente](../components/file-monitor.md). Mit dieser Komponente führt WorkerMan automatisch ein Neuladen durch, wenn eine Datei aktualisiert wird, um die neuen Dateien zu laden und somit wirksam zu machen.

### 9. Unterstützung des Ausführens von Unterprozessen als bestimmter Benutzer
Da die Unterprozesse die tatsächlich eingehenden Benutzeranfragen verarbeiten, dürfen sie aus Sicherheitsgründen nicht über zu hohe Berechtigungen verfügen. Deshalb unterstützt WorkerMan die Konfiguration des Benutzers, unter dem die Unterprozesse ausgeführt werden sollen, um den Server sicherer zu machen.

### 10. Unterstützung der dauerhaften Aufbewahrung von Objekten oder Ressourcen
WorkerMan lädt und analysiert eine PHP-Datei nur einmal im Betrieb und hält sie dann im Speicher, wodurch Klassendeklarationen, PHP-Ausführungsumgebung, Symbole usw. nicht wiederholt erstellt und zerstört werden. Dies unterscheidet sich vollständig vom PHP-Mechanismus unter Webcontainern. In WorkerMan bleiben statische Variablen oder globale Variablen im Lebenszyklus eines Prozesses permanent erhalten, es sei denn, sie werden aktiv entfernt. Dies bedeutet, dass Objekte oder Verbindungen, die in globalen Variablen oder statischen Klassenmitgliedern platziert werden, für alle Anfragen im gesamten Lebenszyklus dieses Prozesses wiederverwendet werden können. Beispielsweise kann eine Datenbankverbindung einmal in einem Prozess initialisiert werden, und alle darauf folgenden Anfragen dieses Prozesses können diese Verbindung wiederverwenden, wodurch der Aufwand für häufige Verbindungen, den TCP-Dreifach-Handshake, die Datenbankberechtigungsprüfung und den TCP-Vierfach-Handshake deutlich reduziert wird und die Effizienz der Anwendungsprogramme erheblich verbessert wird.

### 11. Hohe Leistung
Da eine PHP-Datei nach dem Lesen und Analysieren vom Speicher aus direkt verwendet wird, was zu erheblich reduzierten E/A-Vorgängen auf der Festplatte und vielen zeitaufwändigen Prozessen wie Initialisierung von Requests, Erstellung von Ausführungsumgebungen, lexikalisches und syntaktisches Parsen, Kompilierung von Opcodes, Schließung von Anfragen usw. führt, und da WorkerMan nicht auf nginx, apache und andere Container angewiesen ist und somit die Kommunikationsoverheads von nginx und anderen Containern eliminiert werden und in erster Linie Ressourcen dauerhaft aufbewahrt werden können, muss beispielsweise keine Datenbankverbindung jedes Mal initialisiert werden. Daher bietet die Entwicklung von Anwendungen mit WorkerMan eine sehr hohe Leistung.

### 12. Unterstützung von HHVM
Es wird unterstützt, auf der HHVM-Virtualmaschine zu laufen, um die PHP-Leistung zu vervielfachen. Insbesondere in rechenintensiven Geschäftsfällen ist die Leistung sehr gut. Bei einem Vergleichstest unter realen Lasten stellte sich heraus, dass WorkerMan unter HHVM bei fehlender Geschäftsbelastung etwa 30-80% höhere Netzwerkdurchsätze aufweist als bei der Ausführung unter Zend PHP 5.6.

### 13. Unterstützung von verteilten Bereitstellungen

### 14. Unterstützung von daemonisiertem Verhalten

### 15. Unterstützung des Hörens auf mehreren Ports

### 16. Unterstützung von Standard-Ein- und -Ausgangsumlenkungen
