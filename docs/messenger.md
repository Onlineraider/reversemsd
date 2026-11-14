# Messenger

Alle API-Routen in der folgenden Tabelle sind schulbezogen, sofern nicht
anders angegeben. Die Spezifikation belegt Client-Aufrufe und ausgewählte
Modelle, nicht sämtliche Berechtigungs-, Filter- oder Validierungsregeln.

## Chats und Nachrichten

| Methode | Route (relativ zu `/Api/v1.0`) | Request / Body | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| GET | `/App/{school}/Messenger/Categories` | - | `ChatCategory[]`. |
| GET | `/App/{school}/Messenger/Index?searchquery=&category=&...` | Query-Parameter werden vom Client abhängig von Filtern aufgebaut. | `IndexChat[]`. |
| POST | `/App/{school}/Messenger/Chat/Create` | `ChatCreate` | `Chat`. |
| DELETE | `/App/{school}/Messenger/Chat/{chatId}` | - | Erfolgsstatus; Chat löschen. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Mute` | `{ isSilent }` | Status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Pin` | `{ isPinned }` | Status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/MarkAsUnread` | leer/null | Status. |
| GET | `/App/{school}/Messenger/Chat/{chatId}` | - | `Chat`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/Items?page=1&pagesize=25` | - | `ChatMessage[]`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/ItemsBefore/{messageId}` | - | Ältere `ChatMessage[]`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/ItemsAfter/{messageId}?maxitems=1000` | - | Neuere `ChatMessage[]`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/Info` | - | `ChatEdit`; Empfänger und aktuelle Einstellungen. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/Edit` | - | `ChatEdit`. |
| PATCH | `/App/{school}/Messenger/Chat/{chatId}/Edit` | `ChatEdit` | Status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Upload/{tempId}` | Multipart-Feld `file` | Temporärer Attachment-Upload. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Create` | `ChatMessageCreate` | Neue `ChatMessage`. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Forward` | JSON-GUID-Wert der Nachricht | `ChatMessage`. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Delete` | `ChatMessageSelection` | Boolean-artiges JSON-Ergebnis. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Report/{messageId}` | `{ reason }` | Status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/MarkAsRead` | `{ ids: Guid[] }` oder leere Auswahl | Status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Vote/{messageId}` | `{ vote }` | Status. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/State/{messageId}` | - | `ChatMessageState[]`. |
| GET | `/App/{school}/Messenger/Chat/{emptyGuid}/Item/{messageId}` | - | Einzelne `ChatMessage`. |
| GET | `/App/{school}/Messenger/UserInfo` | - | `MessengerUserInfo`; Quota-/Nutzungsfelder. |

## Anhänge

Diese Download- und Bildrouten stehen im Pfad unter `/{school}` (nicht im
`/Api/v1.0/App`-Präfix) und verwenden laut Spezifikation Bearer-Authentifizierung.

| Methode | Route | Antwort / Verwendung |
|---|---|---|
| GET | `/{school}/Messenger/Chat/{chatId}/Download/{attachmentId}` | Attachment-Bytes. |
| GET | `/{school}/Messenger/Chat/{chatId}/image/{attachmentId}?w=128` | Bildvorschau. |

## Sichtbare Modellfelder

- `ChatBase`: `recipients[]`, `authors[]`, `name`, `authorsLimited`.
- `ChatCreate` ergänzt `useExisting` (Standardwert `true`),
  `replyToMessageId` und `forwardedMessageId`.
- `ChatMessageCreate`: `id`, `content`, `tempId`, `replyToMessageId`,
  `forwardedMessageId`.
- `ChatMessageSelection`: `{ ids: Guid[] }`.
- `ChatVote`: `{ vote }`.
- `ChatReport`: `{ reason }`.

Vollständige Feldlisten für `Chat`, `ChatMessage`, `ChatEdit` und weitere
Modelle sind aus der bereitgestellten Spezifikation nicht ableitbar.

## Datenschutz

Nachrichteninhalte, Empfänger, Anhänge und Personenkennungen sind sensible
Daten. Nicht in Logs, Telemetrie oder ungeschützten Diagnoseansichten
speichern. Serverseitige Zugriffsprüfungen sind für jede Chat- und
Attachment-Aktion erforderlich; UI-Flags sind kein Berechtigungsnachweis.
