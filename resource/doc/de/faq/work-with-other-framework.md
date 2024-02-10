# Wie integrieren Sie sich in andere Frameworks

**Frage:**

Wie integriere ich mich in andere MVC-Frameworks (zum Beispiel thinkPHP, Yii usw.)?

**Antwort:**

![workerman-thinkphp](../images/workerman-work-with-thinkphp.png)

Es wird empfohlen, andere MVC-Frameworks wie folgt zu integrieren (am Beispiel von ThinkPHP):

1. ThinkPHP und Workerman sind zwei unabhängige Systeme, die unabhängig voneinander bereitgestellt werden können (auch auf verschiedenen Servern) und sich nicht gegenseitig beeinflussen.

2. ThinkPHP bietet Webseiten zum Rendern und Anzeigen im Browser über das HTTP-Protokoll.

3. Die von ThinkPHP bereitgestellten Seiten initiieren eine WebSocket-Verbindung mit Workerman.

4. Nach der Verbindung senden sie ein Datenpaket an Workerman (beinhaltet Benutzername, Passwort oder einen bestimmten Token), um die WebSocket-Verbindung einem bestimmten Benutzer zuzuweisen.

5. Nur wenn ThinkPHP Daten an den Browser senden muss, wird die Schnittstelle von Workerman aufgerufen, um die Daten zu senden.

6. Andere Anfragen werden weiterhin nach der herkömmlichen HTTP-Methode von ThinkPHP verarbeitet.

**Zusammenfassung:**

Workerman dient als Kanal, um Daten an den Browser zu senden, und die Schnittstelle von Workerman wird nur aufgerufen, wenn Daten an den Browser gesendet werden müssen. Die Geschäftslogik wird vollständig in ThinkPHP umgesetzt.

Für Informationen dazu, wie ThinkPHP die Workerman-Socket-Schnittstelle aufruft, siehe Abschnitt "Häufig gestellte Fragen - Pushen in anderen Projekten" ([push-in-other-project.md](push-in-other-project.md)).

**ThinkPHP unterstützt offiziell Workerman, siehe [ThinkPHP5-Handbuch](https://www.kancloud.cn/manual/thinkphp5/235128)**
