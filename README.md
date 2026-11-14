# reversemsd

Dokumentation der aus dem MeineSchule.digital-Android-Client
(`v2.3.2 (232)`) bekannten HTTP-, Webportal- und SignalR-Schnittstellen.

## Dokumentation

- [API-Übersicht und Inhaltsverzeichnis](docs/index.md)
- [Konventionen, Herkunft und Grenzen](docs/conventions.md)
- [Authentifizierung und Konto](docs/authentication.md)
- [Webportal, Home.InfoPoint und Stundenplan](docs/web-portal.md)
- [Schule, Personen und Empfänger](docs/school-and-recipients.md)
- [Messenger](docs/messenger.md)
- [Kalender, Medien, HIP, Protokolle und Fundsachen](docs/content-modules.md)
- [Abwesenheiten und Nachmittagsbetreuung](docs/absence-and-care.md)
- [Realtime und Push](docs/realtime-and-push.md)
- [Datenmodelle](docs/data-models.md)
- [Sicherheitsbetrachtung](docs/security.md)
- [Einzeldatei-HTML-Beispielclient](examples/index.html)

## Umfang

Die Referenz erfasst die in der bereitgestellten Spezifikation belegten
Schnittstellen. Sie wurde aus dem dekompilierten Client und – ausschließlich
für die Webportal-Routen – aus einem verlinkten Drittprojekt abgeleitet.
Sie ist keine offizielle API-Dokumentation und keine Bestätigung, dass ein
Server jede Route, Funktion oder Berechtigung freischaltet.

Es wurden keine Server angesprochen. Die Dokumentation enthält keine
Zugangsdaten oder Echtdaten und ist nicht als Anleitung für Zugriffe auf
fremde Konten oder Schulinstanzen gedacht. Siehe [Sicherheitsbetrachtung](docs/security.md).
Das HTML-Beispiel umfasst Chatliste, Nachrichten und Chat-Erstellung sowie
Kalender- und Home.InfoPoint/Notenbuch-Ansichten. Es ist ein unbestätigter
Client-Prototyp und setzt eine autorisierte Instanz, gültige App-ID und
gegebenenfalls passende CORS-Freigaben voraus.
