# Gründe für "send_fail" in status

**Phänomen:**

Beim Ausführen des Befehls "status" tritt die Situation "send_fail" auf. Was ist der Grund dafür?

**Antwort:**

In der Regel ist ein "send_fail" kein großes Problem und tritt normalerweise auf, wenn der Client die Verbindung aktiv beendet oder Daten nicht empfangen kann, was zu einem Fehlschlagen des Datenversands führt.

Es gibt zwei Gründe für "send_fail":

1. Wenn beim Aufrufen der "send" Schnittstelle Daten an den Client gesendet werden sollen, jedoch festgestellt wird, dass der Client bereits getrennt ist, wird der "send_fail" Zähler um eins erhöht. Da es sich um eine vom Client initiierte Trennung handelt, ist dies ein normaler Vorgang, der in der Regel ignoriert werden kann.

2. Die Geschwindigkeit, mit der der Server Daten sendet, ist höher als die Geschwindigkeit, mit der der Client sie empfängt, was dazu führt, dass die Daten kontinuierlich im Sendepuffer des Servers (Workerman erstellt für jeden Client einen Sendepuffer) angesammelt werden. Wenn die Größe des Puffers die Grenzwerte überschreitet (TcpConnection::$maxSendBufferSize standardmäßig 1M), werden die Daten verworfen, ein "onError" Ereignis ausgelöst (wenn vorhanden) und der "send_fail" Zähler erhöht.

Zum Beispiel könnte JavaScript nach dem Minimieren des Browsers pausieren, was dazu führt, dass der Browser keine Daten mehr empfängt und die Daten im Puffer lange Zeit angesammelt werden. Wenn der Puffer das Limit überschreitet, wird bei jedem Aufruf von "send" der "send_fail" Zähler um eins erhöht.

**Zusammenfassung:**

Im Allgemeinen müssen Sie sich keine Sorgen um "send_fail" machen, die durch die Trennung des Clients verursacht wird.

Wenn "send_fail" aufgrund des Stopps des Datenempfangs durch den Client auftritt, sollten Sie überprüfen, ob der Client ordnungsgemäß funktioniert.

Wenn die empfangende Datenrate des Clients **kontinuierlich** unter der Sendegeschwindigkeit des Servers liegt, sollten Sie die Geschäftsprozesse optimieren oder die Leistung des Clients verbessern. Wenn die Bandbreite das Senden beeinträchtigt, können Sie erwägen, die Serverbandbreite zu erhöhen.
