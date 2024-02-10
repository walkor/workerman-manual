# Dateiüberwachungskomponente

**Hintergrund:**

Workerman läuft im Speicher und vermeidet das wiederholte Lesen von Festplatten und das wiederholte Interpretieren und Kompilieren von PHP, um die höchste Leistung zu erzielen. Daher müssen nach Änderungen am Geschäftscode manuell neugeladen oder neu gestartet werden, um wirksam zu werden.

Gleichzeitig bietet Workerman einen Dienst zur Überwachung von Dateiaktualisierungen. Dieser Dienst erkennt automatisch Aktualisierungen von Dateien und führt automatisch ein neues Laden der PHP-Dateien durch. Entwickler können es in das Projekt integrieren und zusammen mit dem Projekt starten.

**Download-Links für die Dateiüberwachungsdienste:**

1. Unabhängige Version: https://github.com/walkor/workerman-filemonitor

2. Inotify-abhängige Version: https://github.com/walkor/workerman-filemonitor-inotify (Inotify-Erweiterung ist erforderlich - [Installationsanweisung](https://php.net/manual/zh/book.inotify.php))

**Unterschiede zwischen den beiden Versionen:**

Die Version am Adresse 1 verwendet die Methode der Überwachung der Dateiaktualisierungszeit durch Abfrage jeder Sekunde, um festzustellen, ob die Datei aktualisiert wurde.

Die Version am Adresse 2 nutzt den Linux-Kernelmechanismus [inotify](https://baike.baidu.com/view/2645027.htm), bei dem das System Workerman benachrichtigt, wenn eine Datei aktualisiert wird.

In der Regel reicht es aus, die unabhängige Version ohne Abhängigkeit zu verwenden.

**Verwendung:**

Kopieren Sie das Verzeichnis Applications/FileMonitor in das Verzeichnis Applications Ihres Projekts.

Wenn Ihr Projekt kein Verzeichnis Applications hat, können Sie die Datei Applications/FileMonitor/start.php an einen beliebigen Ort in Ihrem Projekt kopieren und sie im Startskript einbinden.

Der Überwachungsdienst überwacht standardmäßig das Verzeichnis Applications. Wenn eine Änderung erforderlich ist, kann die Variable ```$monitor_dir``` in Applications/FileMonitor/start.php geändert werden. Der Wert von ```$monitor_dir``` sollte ein absoluter Pfad sein.

**Achtung:**

* Das Reload wird nicht von Windows unterstützt und der Überwachungsdienst kann daher nicht verwendet werden.
* Die Dateiüberwachung funktioniert nur im Debug-Modus. Im Daemon-Modus wird keine Dateiüberwachung durchgeführt (siehe Erklärung unten).
* Nur Dateien, die nach dem Ausführen von Worker::runAll geladen werden, können aktualisiert werden, oder besser gesagt, nur Dateien, die in einem onXXX-Callback geladen werden, können aktualisiert werden.

**Warum wird der Daemon-Modus nicht unterstützt?**

Der Daemon-Modus wird in der Regel im Produktionsumfeld verwendet. Bei der Veröffentlichung einer neuen Version in der Produktionsumgebung werden in der Regel mehrere Dateien gleichzeitig veröffentlicht, und es gibt möglicherweise auch Abhängigkeiten zwischen den Dateien. Da das Synchronisieren mehrerer Dateien auf die Festplatte Zeit in Anspruch nimmt, besteht zu einem bestimmten Zeitpunkt das Risiko, dass nicht alle Dateien auf der Festplatte verfügbar sind. Wenn zu diesem Zeitpunkt eine Dateiaktualisierung erkannt und ein Reload durchgeführt wird, besteht das Risiko schwerwiegender Fehler aufgrund von fehlenden Dateien.

Außerdem werden in der Produktionsumgebung manchmal Bugs online lokalisiert. Wenn der Code direkt bearbeitet und gespeichert wird, tritt der Effekt sofort auf. Es kann zu Syntaxfehlern kommen, die dazu führen, dass der Online-Dienst nicht verfügbar ist. Der richtige Ansatz ist es, den Code nach dem Speichern mit ```php -l yourfile.php``` auf Syntaxfehler zu überprüfen und dann erst den Code neu zu laden. 

Wenn ein Entwickler jedoch tatsächlich den Dateiüberwachungs- und automatischen Aktualisierungsdienst im Daemon-Modus verwenden möchte, kann er den Code in Applications/FileMonitor/start.php entsprechend anpassen und die Prüfung von Worker::$daemonize entfernen.
