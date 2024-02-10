# Nicht unterstützte Funktionen

Nicht unterstützte Funktion/Anweisung | Ersatz | Erklärung
----|------|----
pcntl_fork | Vorher festgelegte Prozessanzahl | 
php://input | [`$request->rawBody()`](http/request.md)| Zum Abrufen der Rohdaten von POST-Anfragen im HTTP-Protokoll
exit | return | Verwendung von exit führt zum Beenden des Prozesses. Bei Rückgabewunsch bitte direkt die return-Anweisung verwenden
die | return | Verwendung von die führt zum Beenden des Prozesses. Bei Rückgabewunsch bitte direkt die return-Anweisung verwenden
header cookie sessionbezogene Funktionen | Siehe [`$request`](http/request.md) und [`$response`](http/response.md) Klasse | 
set_time_limit | Keine | Kann nur auf 0 gesetzt werden, ansonsten führt es zum Beenden des Workerman-Prozesses nach Ablauf einer bestimmten Zeit
