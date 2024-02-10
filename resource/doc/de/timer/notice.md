# Hinweise
## Verwendung von Timern
1. Timer können nur in den Callbacks ```onXXXX``` hinzugefügt werden. Globale Timer werden empfohlen, im ```onWorkerStart``` Callback festgelegt zu werden, während Timer für eine bestimmte Verbindung im ```onConnect``` festgelegt werden sollten.

2. Die hinzugefügten zeitgesteuerten Aufgaben werden im aktuellen Prozess ausgeführt (es wird kein neuer Prozess oder Thread gestartet). Wenn eine Aufgabe sehr zeitaufwändig ist (insbesondere Aufgaben mit Netzwerk-I/O), kann dies dazu führen, dass der aktuelle Prozess blockiert wird und andere Aufgaben vorübergehend nicht verarbeitet werden können. Daher ist es ratsam, zeitaufwändige Aufgaben in einem separaten Prozess auszuführen, z. B. indem ein oder mehrere Worker-Prozesse erstellt werden.

3. Wenn der aktuelle Prozess bereits mit anderen Aufgaben beschäftigt ist oder eine Aufgabe nicht innerhalb des erwarteten Zeitrahmens abgeschlossen ist und es Zeit für einen nächsten Ausführungszyklus ist, wird die Ausführung der nächsten Aufgabe warten, bis die aktuelle Aufgabe abgeschlossen ist. Dies kann dazu führen, dass der Timer nicht entsprechend dem erwarteten Zeitintervall läuft. Mit anderen Worten, die Aufgaben des aktuellen Prozesses werden sequenziell ausgeführt, während in mehreren Prozessen die Aufgaben parallel ausgeführt werden.

4. Beachten Sie, dass die Festlegung von zeitgesteuerten Aufgaben in mehreren Prozessen zu Problemen mit paralleler Ausführung führen kann. Zum Beispiel wird der folgende Code 5 Mal pro Sekunde gedruckt:

    ```php
    $worker = new Worker();
    // 5 Prozesse
    $worker->count = 5;
    $worker->onWorkerStart = function(Worker $worker) {
        // 5 Prozesse, jeder mit einem solchen Timer
        Timer::add(1, function(){
            echo "hi\r\n";
        });
    };
    Worker::runAll();
    ```

    Wenn Sie möchten, dass nur ein Prozess den Timer ausführt, finden Sie ein Beispiel in [Timer::add Beispiel 2](add.md).

5. Es kann zu einer Ungenauigkeit von etwa 1 Millisekunde kommen.

6. Timer können nicht über Prozesse hinweg gelöscht werden. Zum Beispiel kann ein in Prozess A festgelegter Timer nicht direkt mit der Timer::del-Schnittstelle in Prozess B gelöscht werden.

7. Die Timer-IDs können zwischen verschiedenen Prozessen möglicherweise übereinstimmen, aber innerhalb desselben Prozesses wird keine Timer-ID wiederholt.

8. Eine Änderung der Systemzeit wirkt sich auf das Verhalten des Timers aus, daher wird nach einer Änderung der Systemzeit ein Neustart empfohlen.
