# Abwesenheiten und Nachmittagsbetreuung

Die folgenden API-Routen sind schulbezogen. Request- und Antwortfelder sind
nur insoweit aufgeführt, wie sie in der bereitgestellten Spezifikation
sichtbar sind.

## Abwesenheitsübersicht und Meldungen

| Methode | Route (relativ zu `/Api/v1.0`) | Request | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| GET | `/App/{school}/Absence/Index/{yyyy-MM-dd}` | - | `AbsenceIndexByDateData`. |
| GET | `/App/{school}/Absence/Report/Index` | - | `AbsenceReportIndexData`. |
| GET | `/App/{school}/Absence/Report/Student/{studentId}` | - | `AbsenceReport[]`. |
| POST | `/App/{school}/Absence/Report/Student/{studentId}` | `AbsenceReport` | Erstellte Meldung. |
| PATCH | `/App/{school}/Absence/Report/Student/{studentId}` | `AbsenceReport` mit `id` | Status. |
| DELETE | `/App/{school}/Absence/Report/Student/{studentId}/{reportId}` | - | Status. |
| POST | `/App/{school}/Absence/Report/Student/{studentId}/{reportId}/Attachment` | Multipart-Feld `file`; App verwendet `report.jpg`. | Status. |
| GET | `/{school}/StudentAbsenceReport/Image/{reportId}?w=1024` | Bearer-Token | Attachment-Bild. |

`AbsenceReport` enthält sichtbar: `id`, `studentId`, `absenceCategoryId`,
`from`, `to`, `fromLesson`, `toLesson`, `isFullDay`, `text`,
`hasAttachment`, `created`, `creatorUserId` und `creatorUserDisplayName`.
Index-Antworten ergänzen `students[]`, `categories[]` und `lessons[]`.

## Nachmittagsbetreuung

| Methode | Route (relativ zu `/Api/v1.0`) | Request | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| GET | `/App/{school}/AfterSchoolCare/Parent/Index` | - | `AfterSchoolCareIndexData`. |
| GET | `/App/{school}/AfterSchoolCare/Parent/{yyyy-MM-dd}` | - | `AfterSchoolCareStudentData[]`. |
| POST | `/App/{school}/AfterSchoolCare/Parent/Student/{studentId}` | `AfterSchoolCareStudentPickupData` | Eine nicht-null Antwort bedeutet laut Client Erfolg. |
| DELETE | `/App/{school}/AfterSchoolCare/Parent/Student/{studentId}/{yyyy-MM-dd}` | - | Status. |

`AfterSchoolCareStudentPickupData` enthält `date`, `optionId`, `choice`,
`memo`, `isFutureDefault`, `isMonDefault` bis `isSunDefault`, `canEdit` und
Modifier-Identitätsfelder. `AfterSchoolCareIndexData` enthält `students[]`,
`pickupOptions[]` und `pickupWeekDays`.

## Datenschutz und Berechtigungen

Diese Module verarbeiten Schüler-, Abwesenheits- und Familieninformationen.
Nur für die authentifizierte Person sichtbare Daten anzeigen und nicht in
Diagnoseprotokolle übernehmen. Serverseitige Autorisierung muss jede
Objektoperation abdecken; Clientfelder wie `canEdit` dürfen nicht als
Berechtigungsbeweis behandelt werden.
