# Linux Kernel Tuning

Um das System für eine höhere Parallelität zu unterstützen, ist es neben der [Installation der Event-Erweiterung](../install/install.md) auch von größter Bedeutung, den Linux-Kernel zu optimieren. Jede folgende Optimierung ist äußerst wichtig und muss unbedingt einzeln durchgeführt werden.

**Parametererklärung:**

> **max-file**: Stellt die Anzahl der Dateigriffe auf Systemebene dar. Es bezieht sich auf das gesamte Betriebssystem, nicht auf einzelne Benutzer.
>
> **ulimit -n**: Steuert die Anzahl der Dateigriffe, die auf Prozessebene geöffnet werden können. Dies betrifft die steuerbaren Dateigriffe des aktuellen Benutzers in der aktuellen Shell und der gestarteten Prozesse.

Überprüfen Sie die Anzahl der auf Systemebene geöffneten Dateigriffe: `cat /proc/sys/fs/file-max`

Öffnen Sie die Datei /etc/sysctl.conf und fügen Sie die folgenden Einstellungen hinzu
```conf
# Diese Einstellung legt die Anzahl der TIME_WAIT-Zustände des Systems fest. Wenn der Standardwert überschritten wird, werden sie sofort bereinigt.
net.ipv4.tcp_max_tw_buckets = 20000
# Definiert die maximale Länge der Warteschlange für jeden Port im System.
net.core.somaxconn = 65535
# Maximale Anzahl von Verbindungsanforderungen, die in einer Warteschlange eingereiht werden können, bevor diese angenommen werden.
net.ipv4.tcp_max_syn_backlog = 262144
# Anzahl der Datenpakete, die in die Warteschlange gestellt werden können, wenn die Rate, mit der Datenpakete von einem Netzwerkinterface empfangen werden, die Rate übersteigt, mit der der Kernel diese verarbeiten kann.
net.core.netdev_max_backlog = 30000
# Diese Option führt dazu, dass Clients in einem NAT-Netzwerk zeitüberschreitungen haben. Empfohlen wird 0. Ab Linux Kernel-Version 4.12 wurde die tcp_tw_recycle-Konfiguration entfernt. Wenn der Fehler "No such file or directory" angezeigt wird, kann dies ignoriert werden.
net.ipv4.tcp_tw_recycle = 0
# Maximale Anzahl von Dateien, die von allen Prozessen im System geöffnet werden können.
fs.file-max = 6815744
# Größe der Firewall-Verbindungsverfolgungstabelle. Hinweis: Wenn die Firewall nicht aktiviert ist, wird möglicherweise der Fehler "net.netfilter.nf_conntrack_max" is an unknown key" angezeigt. Dies kann ignoriert werden.
net.netfilter.nf_conntrack_max = 2621440
net.ipv4.ip_local_port_range = 10240 65000
```
Führen Sie `sysctl -p` aus, um die Änderungen sofort wirksam werden zu lassen.

**Anmerkung:**
Es gibt viele Optionen, die in der Datei /etc/sysctl.conf festgelegt werden können. Andere Optionen können entsprechend den Anforderungen Ihrer Umgebung festgelegt werden.

## Anzahl der offenen Dateien

Legen Sie die Einstellung für die Anzahl der offenen Dateien im System fest, um das Problem ```too many open files``` bei hoher Parallelität zu lösen. Diese Option beeinflusst direkt die maximale Anzahl von Client-Verbindungen, die ein einzelner Prozess aufnehmen kann.

Soft open files ist ein Linux-Systemparameter, der die maximale Anzahl von Dateigriffen beeinflusst, die ein einzelner Prozess im System öffnen kann. Dieser Wert wirkt sich auf die Anzahl der Benutzer-Verbindungen aus, die ein einzelner Prozess in langen Verbindungen wie bei Chat-Anwendungen aufrechterhalten kann. Mit dem Befehl ```ulimit -n``` können Sie den Wert dieses Parameters sehen. Wenn der Wert beispielsweise 1024 beträgt, bedeutet dies, dass ein einzelner Prozess maximal 1024 oder sogar weniger Verbindungen aufrechterhalten kann (da andere Dateigriffe geöffnet sind). Wenn 4 Prozesse gestartet werden, um Benutzer-Verbindungen aufrechtzuerhalten, kann die Gesamtanzahl der Verbindungen für die gesamte Anwendung niemals mehr als 4*1024 betragen, was bedeutet, dass höchstens 4x1024 Benutzer online sein können. Sie können diese Einstellung erhöhen, um mehr TCP-Verbindungen für die Anwendung aufrechtzuerhalten.

**Änderung des Soft open files auf drei Arten:**

Erste Methode: Führen Sie im Terminal den Befehl `ulimit -HSn 102400` aus und starten Sie dann Workerman neu.

Dies ist nur für das aktuelle Terminal gültig. Nach dem Beenden wird die Anzahl der offenen Dateien wieder auf den Standardwert zurückgesetzt.

Zweite Methode: Fügen Sie in die Datei `/etc/profile` am Ende die Zeile `ulimit -HSn 102400` ein. Auf diese Weise wird der Befehl bei jedem Anmelden im Terminal automatisch ausgeführt. Nach der Änderung muss Workerman neu gestartet werden.

Dritte Methode: Um die Änderung der Anzahl der offenen Dateien dauerhaft zu aktivieren, muss die Konfigurationsdatei `/etc/security/limits.conf` geändert werden. Fügen Sie am Ende dieser Datei Folgendes hinzu:

```plaintext
* soft nofile 1024000
* hard nofile 1024000
root soft nofile 1024000
root hard nofile 1024000
```

Diese Methode erfordert einen Neustart des Servers, um wirksam zu werden.
