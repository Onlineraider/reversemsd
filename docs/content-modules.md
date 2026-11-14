# Kalender, Medien, HIP, Protokolle und Fundsachen

Die schulbezogenen JSON-Routen verwenden `/Api/v1.0/App/{school}`. Routen
explizit unter `/{school}/...` liegen außerhalb dieses API-Präfixes. Die
bereitgestellte Spezifikation belegt keine JSON-Erstellungs-API für Kalender-
oder Medieninhalte.

## Kalender

| Methode | Route (relativ zu `/Api/v1.0`) | Request | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| GET | `/App/{school}/Calendar/Index` | - | `Calendar[]`. |
| GET | `/App/{school}/Calendar/Detail/{id}?lang={languageKey}` | - | `CalendarDetail`. |
| POST | `/App/{school}/Calendar/Confirm/{id}` | `{ answer: number }` | Status. |

## Medien

| Methode | Route (relativ zu `/Api/v1.0`) | Request | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| GET | `/App/{school}/Media/Categories` | - | `IdDisplayName[]`. |
| GET | `/App/{school}/Media/Index?{filters}` | Filter-Query-String des Clients. | `IndexMedia[]`. |
| GET | `/App/{school}/Media/Detail/{id}?lang={languageKey}` | - | `MediaDetail`. |
| POST | `/App/{school}/Media/Confirm/{id}` | Leeres `CalendarConfirm`-Objekt. | Status. |
| POST | `/App/{school}/Media/ToggleLike/{id}` | Leeres `CalendarConfirm`-Objekt. | Status. |
| GET | `/{school}/Media/Image/{mediaId}?w=128` | Bearer-Token | Bildvorschau. |
| GET | `/{school}/Media/Download/{attachmentId}` | Bearer-Token | Attachment-Bytes. |

Die native App öffnet zur Erstellung/Bearbeitung authentifizierte
Webseitenrouten, nicht einen in der Quelle belegten JSON-Endpunkt.

## Home.InfoPoint (HIP)

| Methode | Route (relativ zu `/Api/v1.0`) | Beobachtete Antwort |
|---|---|---|
| GET | `/App/{school}/HIP/Index` | `HIPIndexData { students[] }`. |
| GET | `/App/{school}/HIP/Student/{studentId}` | `HIPStudentData { content, contentCompact }`; beide Werte sind HTML-Fragmente. |

HTML-Fragmente vor einer Darstellung strikt bereinigen. Hinweise zur
Portal-HTML-Referenz befinden sich unter [Webportal](web-portal.md).

## Protokolle

| Methode | Route | Beobachtete Antwort / Verwendung |
|---|---|---|
| GET | `/Api/v1.0/App/{school}/Protocol/Index` | `IndexProtocol[]`. |
| GET | `/Api/v1.0/App/{school}/Protocol/Detail/{id}` | `ProtocolDetail`. |
| GET | `/{school}/Protocol/{protocolId}/download/{attachmentId}` | Bearer-Token; Attachment-Bytes. |

`ProtocolDetail` erweitert `IndexProtocol` um `agenda`, `text`, `attachments[]`
und `attendees[]`. Für diese App-Version ist kein Aufruf zur Protokollerstellung
oder zum Protokoll-Upload belegt.

## Fundsachen

| Methode | Route (relativ zu `/Api/v1.0`) | Request | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| GET | `/App/{school}/LostAndFound/Index` | - | `LostAndFound[]`. |
| POST | `/App/{school}/LostAndFound/` | `{ name, location }` | Neue `LostAndFound`. |
| PATCH | `/App/{school}/LostAndFound/` | Bearbeitbares Fundsachenobjekt | Status. |
| DELETE | `/App/{school}/LostAndFound/{id}` | - | Status. |
| POST | `/App/{school}/LostAndFound/{id}/Attachment` | Multipart-Feld `file` | Status. |
| GET | `/{school}/LostAndFound/Image/{id}?w=512` | Bearer-Token | Bild. |

Die sichtbaren `LostAndFound`-Felder sind `id`, `name`, `location`, `text`,
`creatorUserId`, `found`, `hasAttachment`, `canEdit` und `isActive`.
`canEdit` ist ein Anzeigehinweis und keine serverseitige Autorisierung.
