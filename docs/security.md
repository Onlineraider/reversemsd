# Sicherheitsbetrachtung

Diese Seite fasst statische Client-Befunde und Schutzmaßnahmen aus der
bereitgestellten Spezifikation zusammen. Sie ist keine aktive Prüfung eines
Schulservers und bestätigt keine serverseitige Schwachstelle.

## Beobachtete Client-Risiken und Gegenmaßnahmen

| Einstufung | Beobachtung | Empfehlung |
|---|---|---|
| Hoch | Eine frühere Version des HTML-Chatclients legte das Bearer-Token in `sessionStorage` ab; Browser-Tokens sind ein attraktives XSS-Ziel. | Nur Origin und Schul-Tag dauerhaft speichern. JWT im Arbeitsspeicher halten und nach Neuladen erneut anfordern. Strikte CSP, keine Drittanbieter-Skripte. |
| Mittel | Diagnose- und Rohantwortansichten können Schüler-, Familien-, Nachrichten- und Attachment-Metadaten in einem gemeinsam genutzten Browserprofil offenlegen. | Diagnose lokal halten, Rohdaten in Produktion redigieren/deaktivieren, nicht exportieren und bei Abmeldung leeren. |
| Mittel | Die One-Time-Token-Web-URL enthält das Token als Query-Parameter. Browserhistorie, Logs oder Referrer können es offenlegen. | Einmalig nutzbar, kurzlebig und zielgebunden; `Referrer-Policy: no-referrer`; Query-Token nach Annahme durch eine HttpOnly-Session ersetzen. |
| Mittel | Das Webportal verwendet Cookie-Session und CSRF-Token. Unsichere Redirects oder skriptzugängliche Cookies schwächen den Schutz. | HttpOnly-, Secure- und SameSite-Cookies; Formularziel und Redirect-Origin prüfen; Cookies nicht für JavaScript verfügbar machen. |
| Mittel | Die native App stellt Home.InfoPoint-HTML in einem WebView dar; ein sichtbarer Sanitizer ist im beschriebenen Ablauf nicht vorhanden. | Skripte, Event-Handler und unsichere URLs entfernen; strikten Sanitizer oder isoliertes iframe einsetzen. |
| Niedrig | Von `School/Index` gelieferte Schul-URLs werden laut Client ohne sichtbare Scheme-Prüfung an den Plattformbrowser gegeben. | Nur begründete `https:`-Ziele (gegebenenfalls `http:` in Entwicklung) zulassen und Zielhost anzeigen. |
| Niedrig | Der native Downloader verwendet den `Content-Disposition`-Dateinamen in `Path.Combine` ohne sichtbare Normalisierung. | Verzeichnistrenner und absolute/drive-qualifizierte Namen zurückweisen; lokale Dateinamen erzeugen und app-private Ziele verwenden. |
| Informativ | Im dekompilierten Material wurde weder Certificate-Pinning noch eine Android-`networkSecurityConfig` gefunden. Das ist allein keine Schwachstelle; normale OS-TLS-Prüfung gilt weiterhin. | HTTPS erzwingen, Plattform-Truststore aktuell halten; Pinning nur mit geplantem Schlüsselwechselbetrieb einführen. |

## Serverseitig zu bestätigende Schutzanforderungen

Das Clientmaterial belegt nicht, ob der Server bei jedem Objektzugriff
Zugehörigkeit, Eigentümerschaft und Schulmitgliedschaft prüft. Für eine
autorisierte Sicherheitsprüfung sind ausschließlich ausdrücklich freigegebene
Testumgebungen und Testkonten zu verwenden. Keine Echtdaten und keine fremden
Konten verwenden. Erwartet werden verweigerte Zugriffe ohne Offenlegung, ob
ein nicht berechtigtes Objekt existiert.

Zu verifizieren sind insbesondere:

- Objektbezogene Autorisierung bei Nachrichten, Chat-Anhängen,
  Abwesenheitsmeldungen, Schülerdaten, HIP-Inhalten, Protokollen,
  Medienanhängen und Fundsachenbildern.
- Zulässigkeit von Empfängerregeln bei der Chat-Erstellung.
- Eigentümer-/Berechtigungsprüfung bei Änderungen und Löschungen.
- Einmaligkeit, Ablauf und Zielbindung von One-Time-Tokens.
- Konto- und Gerätebindung bei Push-Installationen.

Diese Punkte sind Prüfanforderungen, keine bestätigten Schwachstellen. Die
Dokumentation enthält bewusst keine Anleitung, fremde IDs oder Konten
auszuprobieren.

## Mindestkontrollen für alternative Clients

- Strikte Content Security Policy; keine `unsafe-inline`-Skripte und keine
  Drittanbieter-Analytics auf authentifizierten Seiten.
- Tokens nur im Arbeitsspeicher oder in einer serverseitigen,
  HttpOnly-/SameSite-geschützten Session; niemals in URLs, `localStorage`,
  Crashreports oder Telemetrie.
- `Authorization`, Passwörter, Token, Nachrichtentexte, Attachment-Namen und
  Personenkennungen aus Logs redigieren.
- Destruktive Aktionen ausdrücklich bestätigen lassen und Fehlerdaten nicht
  roh in gemeinsam genutzten Oberflächen anzeigen.
- Uploadgröße, MIME-Type, Dateiname und Dateiinhalt serverseitig validieren;
  Dateien vor Freigabe prüfen.
- Objektberechtigungen auf dem Server durchsetzen. Felder wie `canEdit`,
  `isOwner`, Rollen oder Capability-Flags sind keine Autorisierungsbeweise.
- HIP-HTML strikt bereinigen; von der Schule gelieferte URLs vor dem Öffnen
  validieren.
