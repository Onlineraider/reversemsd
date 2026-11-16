# Datenmodelle

Diese Referenz enthält die in der bereitgestellten Spezifikation ausdrücklich
genannten Felder. Sie ist keine vollständige Schema-Datei: optionale Felder,
Datentypen, Nullbarkeit und weitere Properties sind nicht immer bekannt.
JSON-Namen folgen im Regelfall camelCase. Bei nicht aufgeführten Modellen
verweist die jeweilige Modulseite auf die bekannte Antwortklasse.

## Authentifizierung und Profil

| Modell | Sichtbare Felder / Bedeutung |
|---|---|
| `TokenResultModel` | Erfolg enthält `token`, `refreshToken`, `expireIn`; 401 kann `twoFactorNeeded` enthalten. |
| `Profile` | `userId`, `displayName`, `schools[]`. |
| `ProfileSchoolInformation` | Mindestens `name`, `tag`. |
| `OneTimeToken` | `token`. |

## Schule, Personen und Empfänger

| Modell | Sichtbare Felder / Bedeutung |
|---|---|
| `School` | `name`, `roles[]`, `features[]`, `students[]`, `links[]`, `translationLanguages[]`, `userHasHIPAccess`, `canUseMessenger`, `canCreateAbsences`, `canCreateLostAndFound`, `canCreateProtocol`, `canCreateMediaItems`, `canCreateCalendarItems`, `hasStudentProperties`. |
| `Link` | `title`, `url`, `appMode`; Werte für `appMode`: `Disabled`, `OpenExternal`, `OpenInternal`. |
| `Student` | `id`, `displayName`, `studentClass`. |
| `StudentInfo` | `id`, `displayName`, `studentClass`, `userId`, `userDisplayName`, `parents`, `partners`, `studentClassInfo`, `studentGroupInfo`, `subjectInfo`, `studentProperties[]`, `hasHIP`. |
| `StudentInfoDetail` | Kann `displayName` und `persons[]` enthalten. Im Fachkontext ist `displayName` die Fachbezeichnung und `persons[]` enthält die zugeordneten Lehrkräfte. |
| `StudentInfoPerson` | `id`, `displayName`, `type`, `userId`, `userDisplayName`. |
| `studentProperties[]` | `name`, `type`, `value`, `mediaExtension`, `mediaSize`, `mediaFileName`, `modified`, `canEdit`. |
| `User` | In der Antwort von `/App/App/Users`: `id`, `displayname`, `hasImage`. |
| `UserInfo` | `id`, `relations[]`; zusätzliche Relations-Strings. |
| `RecipientIndexData.KeyText` | `key`, `text`, `information`, `source`. |
| `SchoolState` | `unreadChatItems[]`, `notConfirmedCalenderItems[]`, `notConfirmedMediaItems[]`, `unreadChatMessageCount`; `newMessages[]` ist veraltet. |

Die native Oberfläche bindet die Fachlehrkräfte an
`subjectInfo.details[]`: Jeder Detail-Eintrag bildet ein Fach und dessen
`persons[]` ab. Eine flache Liste `subjectInfo.persons` ist dafür nicht
maßgeblich und kann fehlen oder leer sein. Die Eigenschaften eines Schülers
werden im Client angezeigt; für ihre Bearbeitung ist der Web-Editor unter
`/{school}/explorer/studentproperties/details/{studentId}` belegt, nicht ein
mobiler JSON-Schreibendpunkt. Die Empfänger-Key-Formate sind in
[Schule und Empfänger](school-and-recipients.md) beschrieben. IDs und
Personendaten nur entsprechend der Serverantwort für den authentifizierten
Benutzer nutzen.

## Messenger

| Modell | Sichtbare Felder / Bedeutung |
|---|---|
| `ChatBase` | `recipients[]`, `authors[]`, `name`, `authorsLimited`. |
| `ChatCreate` | `ChatBase`-Felder plus `useExisting` (Standard `true`), `replyToMessageId`, `forwardedMessageId`. |
| `ChatMessageCreate` | `id`, `content`, `tempId`, `replyToMessageId`, `forwardedMessageId`. |
| `ChatMessageSelection` | `{ ids: Guid[] }`. |
| `ChatVote` | `{ vote }`. |
| `ChatReport` | `{ reason }`. |
| `NotifyHubNewMessage` | Mindestens `chatId` und Nachrichten-`id`. |

`Chat`, `ChatMessage`, `ChatEdit`, `ChatMessageState`, `ChatCategory`,
`IndexChat` und `MessengerUserInfo` werden als Antwortmodelle verwendet; die
bereitgestellte Spezifikation enthält keine vollständigen Feldlisten.

## Inhalte und HIP

| Modell | Sichtbare Felder / Bedeutung |
|---|---|
| `ProtocolDetail` | Erweitert `IndexProtocol` um `agenda`, `text`, `attachments[]`, `attendees[]`. |
| `LostAndFound` | `id`, `name`, `location`, `text`, `creatorUserId`, `found`, `hasAttachment`, `canEdit`, `isActive`. |
| `HIPIndexData` | `students[]`. |
| `HIPStudentData` | `content`, `contentCompact`; beide sind HTML-Fragmente. |

`Calendar`, `CalendarDetail`, `CalendarConfirm`, `IndexMedia`, `MediaDetail`,
`IndexProtocol`, `IdDisplayName` und `DeviceInstallation` sind referenzierte
Modelle, deren vollständige Feldlisten nicht vorliegen.

## Abwesenheit und Betreuung

| Modell | Sichtbare Felder / Bedeutung |
|---|---|
| `AbsenceReport` | `id`, `studentId`, `absenceCategoryId`, `from`, `to`, `fromLesson`, `toLesson`, `isFullDay`, `text`, `hasAttachment`, `created`, `creatorUserId`, `creatorUserDisplayName`. |
| `AbsenceIndexByDateData` | Antwortmodell; nicht vollständig aufgeschlüsselt. |
| `AbsenceReportIndexData` | Enthält unter anderem `students[]`, `categories[]`, `lessons[]`. |
| `AfterSchoolCareStudentPickupData` | `date`, `optionId`, `choice`, `memo`, `isFutureDefault`, `isMonDefault` bis `isSunDefault`, `canEdit` und Modifier-Identitätsfelder. |
| `AfterSchoolCareIndexData` | `students[]`, `pickupOptions[]`, `pickupWeekDays`. |

`AbsenceReport[]`, `AfterSchoolCareStudentData[]` und alle hier nicht weiter
beschriebenen verschachtelten Modelle können weitere Felder enthalten.

## Typ- und Kompatibilitätshinweise

- `Guid`-Werte werden als UUID-Strings übertragen.
- Datums-/Zeitfelder sind grundsätzlich ISO-8601; einige Pfadparameter
  verwenden ausdrücklich `yyyy-MM-dd`.
- Eine Feldliste in dieser Dokumentation ist nicht gleichbedeutend mit einem
  abgeschlossenen JSON-Schema.
- Für Änderungen und unbekannte Felder sind die Modulseiten und
  [Konventionen](conventions.md) maßgeblich.
