# API-Referenz

## Themen

| Bereich | Inhalt |
|---|---|
| [Konventionen und Grenzen](conventions.md) | API-Basis-URLs, Authentifizierung, Belegstatus und Quellengrundlage |
| [Authentifizierung und Konto](authentication.md) | Token, Profil, Login-Brücke und Kontoseiten |
| [Webportal und Stundenplan](web-portal.md) | Cookie-Login, Home.InfoPoint, Portal-HTML und Stundenplan-JSON |
| [Schule und Empfänger](school-and-recipients.md) | Schulstatus, Personen-, Schüler- und Empfängerabfragen |
| [Messenger](messenger.md) | Chats, Nachrichten, Anhänge, Reaktionen und Zustände |
| [Kalender, Medien und weitere Inhalte](content-modules.md) | Kalender, Medien, HIP, Protokolle und Fundsachen |
| [Abwesenheiten und Betreuung](absence-and-care.md) | Abwesenheitsmeldungen und Nachmittagsbetreuung |
| [Realtime und Push](realtime-and-push.md) | SignalR-Hub und Push-Installationen |
| [Datenmodelle](data-models.md) | In der Spezifikation sichtbare Felder und Modellbeziehungen |
| [Sicherheitsbetrachtung](security.md) | Statische Client-Befunde, Schutzmaßnahmen und offene Prüfbereiche |
| [Einzeldatei-HTML-Beispielclient](../examples/msd-client.html) | Browserclient für Anmeldung, Chat-Erstellung, Kalender und Home.InfoPoint |

## Lesen der Routenreferenz

- `{origin}` ist der Host der jeweiligen Instanz.
- `{school}` ist der Schul-Tag aus dem Profil, sofern ein Pfad ihn verwendet.
- `{id}`-Platzhalter stehen für die in der Quelle verwendeten IDs; meist sind
  GUIDs gemeint, sofern nicht anders angegeben.
- `-` in der Body-Spalte bedeutet, dass in der Quelle kein Request-Body
  dokumentiert ist. Daraus folgt nicht zwingend, dass der Server jeden
  zusätzlichen Body akzeptiert.
- Routen und Modelle sind nur so weit beschrieben, wie sie in den Quellen
  sichtbar sind. HTTP-Statuscodes, Validierungsregeln, Pagination,
  Berechtigungsregeln und vollständige Schemas sind häufig nicht bekannt.

## Grundsätzliche Hinweise

Die API kann abhängig von Schulkonfiguration und Benutzerrechten anders
reagieren. Fähigkeiten und Flags in Antworten sind keine
Autorisierungsnachweise. Serverseitige Berechtigungsprüfungen sind maßgeblich.
Siehe [Konventionen](conventions.md) und [Sicherheitsbetrachtung](security.md).

Das [HTML-Beispiel](../examples/msd-client.html) ist nicht gegen einen Server
verifiziert. Es benötigt eine für die Instanz gültige App-ID; bei Aufruf als
lokale Datei oder von einem anderen Host kann die Browser-CORS-Policy die
Anfragen verhindern. HIP-Inhalte werden als HTML-Fragmente geliefert und
deshalb im Beispiel vor der Anzeige auf erlaubte Elemente reduziert.
