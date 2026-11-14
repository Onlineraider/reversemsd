# Schule, Personen und Empfänger

Alle Routen in der Tabelle sind schulbezogen, außer wenn als global markiert.
Modelle und Felder sind in der [Datenmodellübersicht](data-models.md)
zusammengefasst.

## Routen

| Methode | Route (relativ zu `/Api/v1.0`) | Request | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| GET | `/App/{school}/School/Index` | - | `School`; zentrale Fähigkeiten- und Übersichtsantwort. |
| GET | `/App/{school}/School/State` | - | `SchoolState`; Benachrichtigungszähler und IDs. |
| POST | `/App/App/Users` | JSON-Array von User-GUIDs | `User[]` mit `{ id, displayname, hasImage }`. Global, nicht schulbezogen. |
| GET | `/App/{school}/School/UserInfo/{userId}` | - | `UserInfo { id, relations[] }`; zusätzlich nur Relations-Strings. |
| GET | `/App/{school}/School/Student/{studentId}` | - | `StudentInfo`, einschließlich verknüpfter Personen und Fachinformationen. |
| GET | `/App/{school}/School/StudentClasses` | - | `IdDisplayName[]`. |
| GET | `/App/{school}/School/StudentGroups` | - | `IdDisplayName[]`. |
| GET | `/App/{school}/School/Students/{scope}/{id}` | - | `Student[]`; `scope` ist `SC` für Klasse oder `SG` für Gruppe. |
| GET | `/App/{school}/School/Recipients` | - | `RecipientIndexData`; Empfänger-Auswahlindex. |
| GET | `/App/{school}/School/SearchRecipients/{urlEncodedTerm}` | - | `RecipientIndexData.KeyText[]`; Empfängersuche. |

## Fähigkeiten und Verlinkungen

`School/Index` kann unter anderem Rollen, Features, Schüler, Links,
Übersetzungssprachen und Fähigkeiten-Flags enthalten. Sichtbare Flags umfassen
`userHasHIPAccess`, `canUseMessenger`, `canCreateAbsences`,
`canCreateLostAndFound`, `canCreateProtocol`, `canCreateMediaItems`,
`canCreateCalendarItems` und `hasStudentProperties`.

Die zurückgegebenen `Link`-Objekte enthalten `title`, `url` und `appMode`.
`appMode` ist `Disabled`, `OpenExternal` oder `OpenInternal`. Eine URL muss
vor dem Öffnen auf erlaubte Schemes geprüft werden. Fähigkeiten- und
`canEdit`-Felder sind UI-Hinweise und ersetzen keine serverseitige
Berechtigungsprüfung.

## Empfängerkennungen

`RecipientIndexData.KeyText` enthält `key`, `text`, `information` und `source`.
`key` ist unverändert zu erhalten. In der Quelle sichtbare Beispiele:

- Direkter Benutzer: `U:USER:{user-guid}=Name`
- Person: `P:{type}:{person-guid}=Name`
- Schüler: `P:STUDENT:{student-guid}=Name`

Diese Formen beschreiben beobachtete Werte, keine frei konstruierbare
Berechtigung. Empfänger sollten ausschließlich aus den Antworten für den
authentifizierten Benutzer übernommen werden.

## Statusfelder

`SchoolState` enthält `unreadChatItems[]`, `notConfirmedCalenderItems[]`,
`notConfirmedMediaItems[]` und `unreadChatMessageCount`. Das Feld
`newMessages[]` ist in den Modellen als veraltet markiert.
