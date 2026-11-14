# Konventionen, Herkunft und Grenzen

## Herkunft

Der mobile API-Teil basiert auf der bereitgestellten Spezifikation zum
dekompilierten MeineSchule.digital-Android-Client `v2.3.2 (232)`. Die
Webportal-Routen und deren HTML-/JSON-Strukturen stammen aus dem dort
referenzierten Drittprojekt
[`rine77/meineschule-digital`](https://github.com/rine77/meineschule-digital/tree/aa3d8bacb42d676dcc82af7064a1862cee9ba69d).
Diese Portal-Referenz ist nicht für jede Schule bestätigt und kann unabhängig
von der mobilen API geändert werden.

Die Dokumentation wurde nicht durch aktive Serverabfragen verifiziert.
Sie beschreibt beobachtete Client-Aufrufe und sichtbare Modellfelder, nicht
die vollständige interne Server-API.

## API-Basen

| Bezeichnung | Basis |
|---|---|
| API | `{origin}/Api/v1.0` |
| Schulbezogene App-Routen | `{origin}/Api/v1.0/App/{schoolTag}` |
| Globale App-Routen | `{origin}/Api/v1.0/App` |
| SignalR-Hub | `{origin}/notifyHub` |
| Webportal | `{origin}` mit Cookie-Authentifizierung |

API-Routen auf den Themen-Seiten sind relativ zu `{origin}/Api/v1.0`, wenn
nicht ausdrücklich als Portalroute oder Download-/Bildroute unter
`/{school}/...` markiert. Schreibweisen und Groß-/Kleinschreibung folgen der
beobachteten Quelle.

## Request- und Datumsformate

- Der offizielle Client sendet JSON in camelCase. Sein Deserializer akzeptiert
  Property-Namen ohne Beachtung der Groß-/Kleinschreibung.
- Authentifizierte API-Aufrufe verwenden
  `Authorization: Bearer {accessToken}` und üblicherweise
  `Accept: application/json`.
- GUIDs werden als UUID-Strings übermittelt.
- Datums-/Zeitwerte sind normalerweise ISO-8601-Zeitstempel. Wo das Datum Teil
  eines Pfads ist, dokumentiert die Quelle explizit `yyyy-MM-dd`.
- Vom Server gelieferte Werte und Felder können je nach Version und Instanz
  abweichen; unbekannte Eigenschaften sollten nicht verworfen werden, wenn
  Vorwärtskompatibilität wichtig ist.

## Belegstatus

- **Im Client aufgerufen**: Der konkrete Request ist in der App-Spezifikation
  aufgeführt.
- **Modell beobachtet**: Modellname oder Felder sind im Clientmaterial sichtbar.
- **Portal-Referenz**: Route oder Struktur stammt aus dem genannten
  Drittprojekt, nicht aus einer Bestätigung gegen jede Serverinstanz.
- **Nicht belegt**: Berechtigungen, Servervalidierung, weitere Endpunkte und
  vollständige Antwortschemas dürfen nicht aus UI-Verhalten abgeleitet werden.

## Grenzen

Die Dokumentation behauptet keine Vollständigkeit der serverinternen API.
Insbesondere sind Statuscodes und Fehlerkörper, Grenzwerte, Sortierung,
Filtersemantik, Löschfolgen, Rollenprüfungen und alle optionalen Felder nicht
durchgehend bekannt. Eine leere Antwort oder ein Fehler kann eine fehlende
Funktion ebenso bedeuten wie eine Einschränkung durch die jeweilige Instanz.

Es wurden keine Schulserver getestet. Endpunkte mit Schüler-, Familien-,
Nachrichten- oder Anwesenheitsdaten dürfen nur in autorisierten Umgebungen und
mit Testkonten verwendet werden.
