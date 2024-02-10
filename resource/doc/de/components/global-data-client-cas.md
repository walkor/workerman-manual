# cas
**``` (Erfordert Workerman-Version >= 3.3.0) ```**
```php
bool \GlobalData\Client::cas(string $key, mixed $old_value, mixed $new_value)
```
Atomarer Austausch, ersetzt ```$old_value``` durch ```$new_value```.
Der Wert kann nur geschrieben werden, wenn der Wert des entsprechenden Schlüssels nach dem letzten Lesen des aktuellen Clients nicht von einem anderen Client geändert wurde.

## Parameter

 ``` $key ```

Schlüssel. (zum Beispiel ```$global->abc```, ```abc``` ist der Schlüssel)

 ``` $old_value ```

Alter Wert

 ``` $new_value ```

Neuer Wert

## Rückgabewert
Gibt ```true``` zurück, wenn der Austausch erfolgreich war, andernfalls ```false```.

## Hinweis:
Wenn mehrere Prozesse gleichzeitig auf dieselbe gemeinsame Variable zugreifen, müssen mögliche Nebenläufigkeitsprobleme berücksichtigt werden.

Beispiel: Zwei Prozesse A und B fügen gleichzeitig ein Mitglied zur Benutzerliste hinzu.
Die Benutzerliste ist für die Prozesse A und B zu diesem Zeitpunkt ```$global->user_list = array(1,2,3)```.
Prozess A bearbeitet die Variable ```$global->user_list``` und fügt einen Benutzer (4) hinzu.
Prozess B bearbeitet die Variable ```$global->user_list``` und fügt einen Benutzer (5) hinzu.
Prozess A setzt die Variable ```$global->user_list = array(1,2,3,4)``` erfolgreich.
Prozess B setzt die Variable ```$global->user_list = array(1,2,3,5)``` erfolgreich.
In diesem Fall überschreibt die von Prozess B gesetzte Variable die von Prozess A gesetzte Variable und führt zum Datenverlust.

Da das Lesen und Setzen keine atomaren Operationen sind, entstehen dabei Nebenläufigkeitsprobleme.
Um dieses Problem zu lösen, kann die Atomarer Austausch-Schnittstelle verwendet werden.
Vor der Änderung des Werts überprüft die cas-Schnittstelle, ob dieser Wert von anderen Prozessen geändert wurde.
Wenn dies der Fall ist, wird der Austausch nicht durchgeführt und ```false``` zurückgegeben. Andernfalls wird der Austausch durchgeführt und ```true``` zurückgegeben.
Siehe das folgende Beispiel.

 **Hinweis:** 
In manchen Fällen ist es unproblematisch, wenn gemeinsame Daten von gleichzeitigen Prozessen überschrieben werden, zum Beispiel das derzeit höchste Angebot in einem Auktionssystem, oder der aktuelle Bestand eines bestimmten Produkts.

## Beispiel

```php
$global = new GlobalData\Client('127.0.0.1:2207');

// Liste initialisieren
$global->user_list = array(1,2,3);

// Atomar einen Wert zur user_list hinzufügen
do
{
    $old_value = $new_value = $global->user_list;
    $new_value[] = 4;
}
while(!$global->cas('user_list', $old_value, $new_value));

var_export($global->user_list);
```
