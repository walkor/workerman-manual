# Schnittstellen bereitgestellt von der Connection-Klasse

In Workerman gibt es zwei wichtige Klassen Worker und Connection.

Jede Client-Verbindung entspricht einem Connection-Objekt. Es ist möglich, Callbacks wie onMessage und onClose für das Objekt festzulegen. Außerdem bietet die Connection-Klasse Schnittstellen zum Senden von Daten an den Client (send) und zum Schließen der Verbindung (close) sowie andere notwendige Schnittstellen.

Man kann sagen, dass Worker ein Überwachungsbehälter ist, der dafür zuständig ist, Client-Verbindungen zu akzeptieren und sie als Connection-Objekte bereitzustellen, um sie vom Entwickler manipulieren zu lassen.
