# Fehler beim Beenden

## Phänomen:
Die Ausführung von `php start.php stop` zeigt `stop fail` an.

### Erste Möglichkeit
Voraussetzung ist, dass Workerman im Debug-Modus gestartet wurde und der Entwickler in der Terminal `ctrl z` gedrückt hat, um das `SIGSTOP`-Signal an Workerman zu senden, was dazu führte, dass Workerman in den Hintergrund verschoben und angehalten wurde, so dass es nicht auf das Stop-Befehl (`SIGINT`-Signal) reagieren kann.

**Lösung:**
Geben Sie im Terminal, in dem Workerman gestartet wurde, `fg` ein (um das `SIGCONT`-Signal zu senden), drücken Sie die Eingabetaste, um Workerman in den Vordergrund zu bringen, und drücken Sie `ctrl c` (um das `SIGINT`-Signal zu senden) und Workerman zu stoppen.

Wenn das Stoppen nicht möglich ist, versuchen Sie die folgenden beiden Befehle auszuführen:
```bash
killall -9 php
```
```bash
ps aux|grep -i workerman|awk '{print $2}'|xargs kill -9
```

### Zweite Möglichkeit
Der Benutzer, der das Stoppen ausführt, und der Benutzer, der Workerman gestartet hat, sind nicht identisch, dh der Stop-Benutzer hat keine Berechtigung, Workerman zu stoppen.

**Lösung:**
Wechseln Sie zum Benutzer, der Workerman gestartet hat, oder verwenden Sie einen Benutzer mit höheren Berechtigungen, um Workerman zu stoppen.

### Dritte Möglichkeit
Die PID-Datei des Workerman-Hauptprozesses wurde gelöscht, was dazu führt, dass das Skript den Prozess-PID nicht finden kann und das Beenden fehlschlägt.

**Lösung:**
Speichern Sie die PID-Datei an einem sicheren Ort, siehe Handbuch [Worker::$pidFile](../worker/pid-file.md).

### Vierte Möglichkeit
Der Prozess, der der PID-Datei des Workerman-Hauptprozesses entspricht, ist nicht der Workerman-Prozess.

**Lösung:**
Öffnen Sie die PID-Datei des Workerman-Hauptprozesses und überprüfen Sie die Hauptprozess-PID. Die PID-Datei befindet sich standardmäßig im Workerman-Verzeichnis. Führen Sie den Befehl `ps aux | grep Hauptprozess-PID` aus, um zu überprüfen, ob der entsprechende Prozess ein Workerman-Prozess ist. Wenn nicht, liegt es möglicherweise daran, dass der Server neu gestartet wurde, was dazu führt, dass die von Workerman gespeicherte PID veraltet ist und diese PID zufällig von einem anderen Prozess verwendet wird, was das Beenden fehlschlagen lässt. 
Falls dies der Fall ist, löschen Sie die PID-Datei.

### Fünfte Möglichkeit
Die grpc-Erweiterung ist installiert, aber es wurden keine entsprechenden Umgebungsvariablen für die grpc-Erweiterung festgelegt, was dazu führt, dass beim Start ein zusätzlicher Fork-Prozess erstellt wird, der das Beenden fehlschlagen lässt.
