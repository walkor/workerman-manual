# Die Worker-Klasse
In WorkerMan gibt es zwei wichtige Klassen: Worker und Connection.

Die Worker-Klasse wird verwendet, um den Port zu überwachen und Callback-Funktionen für Ereignisse wie das Herstellen einer Verbindung zum Client, das Empfangen von Nachrichten und das Trennen der Verbindung zu setzen, um die Geschäftslogik umzusetzen.

Es ist möglich, die Anzahl der Prozesse der Worker-Instanz (count-Attribut) festzulegen. Die Worker-Hauptprozess erstellt count Unterprozesse, die gleichzeitig denselben Port überwachen, um Client-Verbindungen entgegenzunehmen und Ereignisse zu bearbeiten.
