# MeineSchule.digital API specification

## Scope and evidence

This document describes the HTTP and SignalR interfaces referenced by
`MeineSchule.digital_v2.3.2 (232)`. It was derived from the decompiled
application source in this repository, not from server probing. A server may
hide a feature or reject a request according to school configuration and the
authenticated user's permissions.

`Verified in app` means the exact request is issued by the app. `Observed
model` means its request or response fields are visible in the bundled C# model
classes. Do not assume undocumented endpoints, role checks, or write access.

## Conventions

* API root: `{origin}/Api/v1.0`.
* School-scoped app root: `{origin}/Api/v1.0/App/{schoolTag}`.
* Global app root: `{origin}/Api/v1.0/App`.
* JSON is camelCase in the official client, but its deserializer accepts
  case-insensitive property names. A browser client should accept both forms.
* Authenticated requests use `Authorization: Bearer {accessToken}` and normally
  `Accept: application/json`.
* GUID values are UUID strings. Date values are ISO-8601 JSON timestamps unless
  an endpoint explicitly puts `yyyy-MM-dd` in its path.
* All server-side permission checks remain authoritative. UI flags are only
  presentation hints.

## Authentication and account routes

| Method | Path | Request | Response / use |
|---|---|---|---|
| POST | `/Token/CreateToken` | `{ userName, password, twoFactorCode: string|null, appId }` | `TokenResultModel`; success includes `token`, `refreshToken`, `expireIn`. A 401 response can contain `twoFactorNeeded`. |
| POST | `/Token/RefreshToken` | `{ token, refreshToken, appId }` | refreshed token pair. Official app refreshes roughly two minutes before access-token expiry. |
| GET | `/App/App/Profile` | Bearer auth | `Profile { userId, displayName, schools[] }`. |
| POST | `/App/App/OneTimeToken` | empty body, Bearer auth | `OneTimeToken { token }`; use to enter web UI. |
| POST | `/Token/MSDAppExternalLogin/{scheme}/{appId}` | browser-auth redirect flow | external-provider login; app callback is `msdapp://`. |
| POST | `/Token/MSDAppExternalAppleLogin/{appId}` | `{ idToken }` | Apple platform flow. |
| GET | `/Account/Image/{userId}?w={pixels}` | Bearer auth | user avatar image. This is **not** school-scoped. The app requests width 48; clients can request another supported width. |
| GET | `/account/forgotpassword` | browser page | password-recovery web page. |
| GET | `/account/delete` | browser page | account-deletion web page. |
| GET | `/Account/Login?token={oneTimeToken}&inapp=1&returnurl=/{schoolTag}/{path}` | browser page | authenticated web UI bridge. Used for profile editing, schedule, media/calendar authoring, and student properties. |

`ProfileSchoolInformation` contains at least `name` and `tag`. The native app
stores the selected tag locally as `Auth.School`; switching it reloads school
state. Never log or persist access/refresh tokens in plain text.

## Web portal: Home.InfoPoint and individual timetable

This is a separate, cookie-authenticated web interface, not the mobile
`/Api/v1.0` interface above. The routes and fields in this section are derived
from the public, third-party reference client
[`rine77/meineschule-digital`](https://github.com/rine77/meineschule-digital/tree/aa3d8bacb42d676dcc82af7064a1862cee9ba69d).
They have not been confirmed against every school, and portal HTML can change
independently from the mobile app.

| Method | Path | Request | Response / use |
|---|---|---|---|
| GET | `/account/login` | cookie jar enabled | HTML login form containing hidden `__RequestVerificationToken`. |
| POST | `/account/login` | form fields `UserName`, `Password`, `RememberMe=false`, `__RequestVerificationToken` | authenticated portal session cookie on success. Detect a returned password form as failed login. |
| GET | `/{schoolSlug}/hip` | authenticated portal cookie | rendered Home.InfoPoint HTML. |
| GET | `/{schoolSlug}/schedule` | authenticated portal cookie | timetable page; selected student is an option in `#Selection` with value `s:{studentId}`. |
| GET | `/{schoolSlug}/schedule?handler=Schedule&date={YYYY-MM-DD}&cog=&t=&s={studentId}&view=` | authenticated portal cookie | JSON for the timetable week containing `date`. `cog` is a class/group **name**; empty `view` returns the substitution overview. See below. |

Portal login is protected by an antiforgery token and relies on cookies; it is
not interchangeable with sending a mobile bearer JWT to these routes. Preserve
the cookie jar and origin, follow only same-origin redirects, and treat an
`/account/accessdenied` redirect or returned login page as an authorization
failure.

### Home.InfoPoint portal document structure

The reference parser expects `.main.withschoolmenu`, an `h2` student name, and
German section headings. The documented data extraction is:

* `Noten`: every subject is an `h4`; following table rows with four cells map
  to `date`, `value`, `remark`, and `information`.
* `Fehlzeiten`: table rows with two cells map to `date` and `remark`.
* `Bemerkungen`: rows with five cells map to `date`, `text`, `subject`, `type`,
  and `author`.
* `Unterricht`: rows with three cells map to `date`, `subject`, and `homework`.

Dates are displayed as `DD.MM.YYYY`. A phrase matching `zum DD.MM.YYYY` in a
homework text is an explicit due-date convention, not a server-provided field.
Any remaining due-date inference from the next non-cancelled matching timetable
lesson is client logic and must be labelled as such.

### Timetable JSON structure

Confirmed against live responses (debug logs of a native client). The handler returns:

```json
{
  "startTimes": ["08:00:00", "..."],          // lesson start times; no end times (client assumes +45 min)
  "infos": [{"date": "2026-11-19T00:00:00", "informationPublic": "Präsenz: Lie", "informationTeachers": null, "isActive": true}],
  "schedule": {"days": [{"d": "2026-11-19T00:00:00", "l": [ /* one slot per lesson, zero-based */ ]}]},
  "replacements": [ /* substitution rows, see below */ ],
  "classesOrGroups": [ ... ], "teachers": [ ... ]   // selector data (names), shape varies
}
```

A slot in `l` is an array of entries or an object `{ "s": [], "e": [entries] }`. Entry fields:

| Field | Meaning |
|---|---|
| `st` | subject short name |
| `ct` | class(es) text |
| `gt` | group text |
| `tt` | teacher short name(s) |
| `rm` | room |
| `txt` | free text |
| `rix` | index into the response's `replacements[]`; `-1` = regular lesson |
| `cancelled` | `true` for cancelled lessons |
| `sc`, `tc` | internal ids/codes, not for display |

Do not assume the slot index equals the human-visible lesson number.

#### Query parameters of `handler=Schedule`

* `date` selects the week.
* `s={studentId}` selects a student's plan.
* `cog={name}` selects a class or group plan. **The value is the class/group
  *name* (e.g. `10c`, `En1_10_8b`), not an id.** With an id the server answers
  `schedule: null`.
* `t={teacher}` selects a teacher's plan.
* `view=` (empty) returns the substitution overview (`replacements` for the
  whole school, without a personal `schedule`). The portal's "Vertretungen"
  selector entry corresponds to this empty selection.

#### `replacements[]` row

| Field | Meaning |
|---|---|
| `id`, `foreignId`, `scheduleForeignId`, `scheduleId` | identifiers |
| `classesOrGroups` / `newClassesOrGroups` | affected class/group text (old / new) |
| `studentClassIds[]`, `studentGroupIds[]` (+ `new…`) | ids of the affected classes/groups |
| `date`, `lessonIndex`, `lessons` | original day, first lesson (1-based) and number of consecutive lessons (`null` = 1) |
| `newDate`, `newLessonIndex` | target day/lesson when the lesson is moved (`null` = unchanged) |
| `subject`/`subjectId` → `newSubject`/`newSubjectId` | subject before / after |
| `teachers`/`teacherIds[]` → `newTeachers`/`newTeacherIds[]` | teacher text before / after (e.g. `"Löc"` → `"Löc, Sra"`) |
| `rooms`/`roomId` → `newRooms`/`newRoomId` | room before / after |
| `reason` (number), `reasonText` | e.g. `1` = `Änd.`, `3` = `Verl.` (moved); other values such as "Entf." (cancelled) are shown verbatim |
| `message`, `messageTeacher`, `text` | free texts (`message` is visible to students) |

Rendering rule used by the portal: a moved lesson is shown as
`3-4 (19.11.2026) > 6-7 (23.11.2026)`; changed fields as `old > new`. A row
without any `new…` value and without a move is a cancellation. A lesson entry
with `rix >= 0` belongs to `replacements[rix]` of the same response, which gives
the old values for highlighting changes in the regular plan.

## School, people, links, and state

| Method | Path | Request | Response / use |
|---|---|---|---|
| GET | `/App/{school}/School/Index` | - | `School`. The central capability and overview response. |
| GET | `/App/{school}/School/State` | - | `SchoolState` notification counters/IDs. |
| POST | `/App/App/Users` | JSON array of user GUIDs | `User[]`; each `{ id, displayname, hasImage }`. Global app route, not school-scoped. |
| GET | `/App/{school}/School/UserInfo/{userId}` | - | `UserInfo { id, relations[] }`; extra relation strings only. |
| GET | `/App/{school}/School/Student/{studentId}` | - | `StudentInfo`, including linked people and subject info. |
| GET | `/App/{school}/School/StudentClasses` | - | `IdDisplayName[]`. |
| GET | `/App/{school}/School/StudentGroups` | - | `IdDisplayName[]`. |
| GET | `/App/{school}/School/Students/{scope}/{id}` | - | `Student[]`; `scope` is `SC` for a class and `SG` for a group. |
| GET | `/App/{school}/School/Recipients` | - | `RecipientIndexData`, the full recipient picker index. |
| GET | `/App/{school}/School/SearchRecipients/{urlEncodedTerm}` | - | `RecipientIndexData.KeyText[]` for recipient search. |

Important response models:

* `School`: `name`, `roles[]`, `features[]`, `students[]`, `links[]`,
  `translationLanguages[]`, `userHasHIPAccess`, `canUseMessenger`,
  `canCreateAbsences`, `canCreateLostAndFound`, `canCreateProtocol`,
  `canCreateMediaItems`, `canCreateCalendarItems`, `hasStudentProperties`.
* `Link`: `title`, `url`, `appMode`: `Disabled`, `OpenExternal`, or
  `OpenInternal`. Validate scheme before opening a returned URL.
* `Student`: `id`, `displayName`, `studentClass`.
* `StudentInfo`: `id`, `displayName`, `studentClass`, `userId`,
  `userDisplayName`, `parents`, `partners`, `studentClassInfo`,
  `studentGroupInfo`, `subjectInfo`, `studentProperties[]`, `hasHIP`.
* Each `StudentInfoDetail` can contain `displayName` and `persons[]`.
  `StudentInfoPerson`: `id`, `displayName`, `type`, `userId`,
  `userDisplayName`.
* **Subject teachers are hierarchical:** the native `MainPage` binds its
  per-subject teacher list to `CurrentStudent.SubjectInfo.Details`, not to a
  flattened `subjectInfo.persons` list. Every detail's `displayName` is the
  subject heading and its `persons[]` are the teachers for that subject. A
  compatible client should render every `SubjectInfo.Details[]` entry, even
  if a flat list is absent or empty.
* `studentProperties[]` contains `name`, `type`, `value`, `mediaExtension`,
  `mediaSize`, `mediaFileName`, `modified`, and `canEdit`. The native model
  exposes these values for display but the recovered client path for editing
  is the authenticated web editor described below; no mobile JSON save route
  was identified in the decompiled app.
* `RecipientIndexData.KeyText`: `key`, `text`, `information`, `source`.
  Preserve `key` verbatim. A direct user recipient looks like
  `U:USER:{user-guid}=Name`; a person recipient looks like
  `P:{type}:{person-guid}=Name`; a student recipient is
  `P:STUDENT:{student-guid}=Name`.
* `SchoolState`: `unreadChatItems[]`, `notConfirmedCalenderItems[]`,
  `notConfirmedMediaItems[]`, `unreadChatMessageCount`; legacy
  `newMessages[]` is marked obsolete in the models.

Web UI pages called by the native app: `/{school}/schedule` and
`/{school}/explorer/studentproperties/details/{studentId}`. It gets an
`Account/Login` one-time-token URL first rather than exposing a JSON write API.
The latter is opened by `MainViewModel.StudentActionAsync("PROPERTIES")` for
the selected student and is the editor reached by the pencil in the supplied
recording. This is a per-student-properties editor; it is **not evidence of a
direct JSON endpoint for changing the account login name**. The separate
account profile editor remains `/{school}/account/profile`.

## Messenger

| Method | Path | Body | Response / use |
|---|---|---|---|
| GET | `/App/{school}/Messenger/Categories` | - | `ChatCategory[]`. |
| GET | `/App/{school}/Messenger/Index?searchquery=&category=&...` | - | `IndexChat[]`; app builds parameters from its filters. |
| POST | `/App/{school}/Messenger/Chat/Create` | `ChatCreate` | `Chat`. |
| DELETE | `/App/{school}/Messenger/Chat/{chatId}` | - | success status; delete a chat. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Mute` | `{ isSilent }` | status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Pin` | `{ isPinned }` | status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/MarkAsUnread` | empty/null | status. |
| GET | `/App/{school}/Messenger/Chat/{chatId}` | - | `Chat`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/Items?page=1&pagesize=25` | - | `ChatMessage[]`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/ItemsBefore/{messageId}` | - | older `ChatMessage[]`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/ItemsAfter/{messageId}?maxitems=1000` | - | newer `ChatMessage[]`. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/Info` | - | `ChatEdit`; recipients and current chat settings. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/Edit` | - | `ChatEdit`. |
| PATCH | `/App/{school}/Messenger/Chat/{chatId}/Edit` | `ChatEdit` | status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Upload/{tempId}` | multipart field `file` | temporary attachment upload. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Create` | `ChatMessageCreate` | new `ChatMessage`. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Forward` | message GUID JSON value | `ChatMessage`. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Delete` | `ChatMessageSelection` | JSON boolean-like result. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Report/{messageId}` | `{ reason }` | status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/MarkAsRead` | `{ ids: Guid[] }` or empty selection | status. |
| POST | `/App/{school}/Messenger/Chat/{chatId}/Vote/{messageId}` | `{ vote }` | status. |
| GET | `/App/{school}/Messenger/Chat/{chatId}/State/{messageId}` | - | `ChatMessageState[]`. |
| GET | `/App/{school}/Messenger/Chat/{emptyGuid}/Item/{messageId}` | - | one `ChatMessage`. |
| GET | `/App/{school}/Messenger/UserInfo` | - | `MessengerUserInfo` quota/usage fields. |
| GET | `/{school}/Messenger/Chat/{chatId}/Download/{attachmentId}` | Bearer auth | attachment bytes. |
| GET | `/{school}/Messenger/Chat/{chatId}/image/{attachmentId}?w=128` | Bearer auth | image preview. |

`ChatBase`: `recipients[]`, `authors[]`, `name`, `authorsLimited`.
`ChatCreate` adds `useExisting` (default true), `replyToMessageId`, and
`forwardedMessageId`. `ChatMessageCreate` has `id`, `content`, `tempId`,
`replyToMessageId`, and `forwardedMessageId`. `ChatMessageSelection` is
`{ ids: Guid[] }`; `ChatVote` is `{ vote }`; `ChatReport` is `{ reason }`.

## Calendar and media

| Method | Path | Body | Response / use |
|---|---|---|---|
| GET | `/App/{school}/Calendar/Index` | - | `Calendar[]`. |
| GET | `/App/{school}/Calendar/Detail/{id}?lang={languageKey}` | - | `CalendarDetail`. |
| POST | `/App/{school}/Calendar/Confirm/{id}` | `{ answer: number }` | status. |
| GET | `/App/{school}/Media/Categories` | - | `IdDisplayName[]`. |
| GET | `/App/{school}/Media/Index?{filters}` | - | `IndexMedia[]`; app passes its filter query string through. |
| GET | `/App/{school}/Media/Detail/{id}?lang={languageKey}` | - | `MediaDetail`. |
| POST | `/App/{school}/Media/Confirm/{id}` | empty `CalendarConfirm` object | status. |
| POST | `/App/{school}/Media/ToggleLike/{id}` | empty `CalendarConfirm` object | status. |
| GET | `/{school}/Media/Image/{mediaId}?w=128` | Bearer auth | image preview. |
| GET | `/{school}/Media/Download/{attachmentId}` | Bearer auth | attachment bytes. |

The native app opens authenticated website routes rather than a JSON authoring
endpoint for calendar and media creation/editing. It does not prove a JSON
create API for those modules.

## Home.InfoPoint, protocols, and lost property

| Method | Path | Body | Response / use |
|---|---|---|---|
| GET | `/App/{school}/HIP/Index` | - | `HIPIndexData { students[] }`. |
| GET | `/App/{school}/HIP/Student/{studentId}` | - | `HIPStudentData { content, contentCompact }`; both are HTML fragments. |
| GET | `/App/{school}/Protocol/Index` | - | `IndexProtocol[]`. |
| GET | `/App/{school}/Protocol/Detail/{id}` | - | `ProtocolDetail`. |
| GET | `/{school}/Protocol/{protocolId}/download/{attachmentId}` | Bearer auth | attachment bytes. |
| GET | `/App/{school}/LostAndFound/Index` | - | `LostAndFound[]`. |
| POST | `/App/{school}/LostAndFound/` | `{ name, location }` | new `LostAndFound`. |
| PATCH | `/App/{school}/LostAndFound/` | editable lost-property object | status. |
| DELETE | `/App/{school}/LostAndFound/{id}` | - | status. |
| POST | `/App/{school}/LostAndFound/{id}/Attachment` | multipart field `file` | status. |
| GET | `/{school}/LostAndFound/Image/{id}?w=512` | Bearer auth | image. |

`ProtocolDetail` extends `IndexProtocol` and adds `agenda`, `text`,
`attachments[]`, `attendees[]`. `LostAndFound` visibly uses `id`, `name`,
`location`, `text`, `creatorUserId`, `found`, `hasAttachment`, `canEdit`, and
`isActive`. The app has no protocol-create or protocol-upload call in this
version.

## Absences and after-school care

| Method | Path | Body | Response / use |
|---|---|---|---|
| GET | `/App/{school}/Absence/Index/{yyyy-MM-dd}` | - | `AbsenceIndexByDateData`. |
| GET | `/App/{school}/Absence/Report/Index` | - | `AbsenceReportIndexData`. |
| GET | `/App/{school}/Absence/Report/Student/{studentId}` | - | `AbsenceReport[]`. |
| POST | `/App/{school}/Absence/Report/Student/{studentId}` | `AbsenceReport` | created report. |
| PATCH | `/App/{school}/Absence/Report/Student/{studentId}` | `AbsenceReport` with `id` | status. |
| DELETE | `/App/{school}/Absence/Report/Student/{studentId}/{reportId}` | - | status. |
| POST | `/App/{school}/Absence/Report/Student/{studentId}/{reportId}/Attachment` | multipart `file`, app uses `report.jpg` | status. |
| GET | `/{school}/StudentAbsenceReport/Image/{reportId}?w=1024` | Bearer auth | attachment image. |
| GET | `/App/{school}/AfterSchoolCare/Parent/Index` | - | `AfterSchoolCareIndexData`. |
| GET | `/App/{school}/AfterSchoolCare/Parent/{yyyy-MM-dd}` | - | `AfterSchoolCareStudentData[]`. |
| POST | `/App/{school}/AfterSchoolCare/Parent/Student/{studentId}` | `AfterSchoolCareStudentPickupData` | non-null response means success. |
| DELETE | `/App/{school}/AfterSchoolCare/Parent/Student/{studentId}/{yyyy-MM-dd}` | - | status. |

`AbsenceReport`: `id`, `studentId`, `absenceCategoryId`, `from`, `to`,
`fromLesson`, `toLesson`, `isFullDay`, `text`, `hasAttachment`, `created`,
`creatorUserId`, `creatorUserDisplayName`. Index responses add `students[]`,
`categories[]`, and `lessons[]`.

`AfterSchoolCareStudentPickupData`: `date`, `optionId`, `choice`, `memo`,
`isFutureDefault`, `isMonDefault` through `isSunDefault`, `canEdit`, and
modifier identity fields. `AfterSchoolCareIndexData` contains `students[]`,
`pickupOptions[]`, and `pickupWeekDays`.

## Realtime and push

* SignalR hub: `{origin}/notifyHub`, bearer token supplied as the SignalR access
  token. The app enables automatic reconnect.
* Hub event: `newmessage(NotifyHubNewMessage)`, containing at least `chatId`
  and message `id`; app refreshes the matching chat.
* Push installation endpoint: `PUT` and `DELETE`
  `/Api/v1.0/App/PushNotifications/Installations`. The JSON body is the native
  `DeviceInstallation` model (platform push channel, installation identifier,
  and tags). This app uses device-local push tokens; a web client should use
  the browser Push API and its own service worker instead of copying mobile
  installation data.

## Implementation notes

1. Use the `School/Index` booleans and response availability to discover what
   to display, but handle 401, 403, 404, and empty lists gracefully.
2. Keep tokens, message bodies, attachments, pupil data, and parent/teacher
   identities out of diagnostics and browser logs.
3. Sanitize HIP HTML before rendering. The app wraps it in a local CSS template;
   a web client should use a sandboxed iframe or a strict sanitizer.
4. Display only people and user IDs returned to the authenticated user. Do not
   fabricate recipient strings or infer missing staff data.
5. For web UI routes, obtain a new one-time token immediately before opening
   the page. Treat it as short-lived and sensitive.

## Security review and verification checklist

This section is a static review of the bundled Android application and the
local HTML client. It is **not** the result of active testing against a school
server. Items labelled `Verify server-side` are requirements to test only with
written authorization and test accounts; they are not confirmed vulnerabilities.

### Confirmed client-side risks and mitigations

| Severity | Finding | Evidence | Mitigation |
|---|---|---|---|
| High | A browser bearer token is an attractive XSS target. | An earlier version of `chat-client/index.html` wrote the token into `sessionStorage`. | The client now persists only origin and school tag. The JWT stays in page memory and is required again after a reload. Keep a restrictive CSP and never add third-party scripts. |
| Medium | Diagnostic and raw-response screens can expose student, family, message, and attachment metadata to anyone using the same unlocked browser profile. | The client explicitly renders API responses for debugging. | Keep diagnostics local, redact/disable raw data in production, never export it, and clear it on logout. |
| Medium | One-time web-login tokens are put in a URL query string by the official app. Such URLs can reach browser history, logs, or referrer headers if the receiving web flow is not careful. | `BackendService.GetAuthendicatedWebsiteUrl` builds `Account/Login?token=...&inapp=1&returnurl=...`. | Make tokens single-use, very short-lived, scope them to the destination, send `Referrer-Policy: no-referrer`, and replace the query token with an HttpOnly session immediately. |
| Medium | The portal login is cookie- and CSRF-token based. A custom web client that ignores origin checks, follows cross-origin redirects, or stores its session cookie in script-accessible storage would weaken those protections. | The independent portal client first GETs `/account/login`, submits `__RequestVerificationToken`, and relies on an `aiohttp` cookie jar. | Keep cookies in an HttpOnly, Secure, SameSite session store; validate the login form action and redirect origin; do not expose portal cookies to browser JavaScript. |
| Medium | The original app renders server-provided Home.InfoPoint HTML in a WebView without a visible sanitizer in the ViewModel. A privileged or compromised content author could turn this into stored WebView script/content injection. | `HIPViewModel.ShowCurrentData` inserts `Content`/`ContentCompact` into `hiptemplate.htm`; `HIPPage` uses a WebView. | Sanitize server HTML, remove scripts/event handlers/unsafe URLs, and use a sandboxed iframe or equivalent. The HTML client applies a sanitizer and empty sandbox. |
| Low | The original app opens school links supplied by `School/Index` through the platform browser without visible URL-scheme validation. | `MainViewModel.PaneAsync` passes `Link.Url` to `Browser.OpenAsync`. | Allow only `https:` (and justified `http:` for local development); display destination host before opening. The HTML client validates `http`/`https`. |
| Low | The native file downloader uses the server-supplied `Content-Disposition` filename in `Path.Combine(folder, filename)` with no visible filename normalization. A malicious server or proxy could attempt path traversal or overwrite behavior. | `BackendService.DownloadFileToFolder`. | Strip directory separators, reject absolute/drive-qualified names, generate a local filename, and use an app-private destination. |
| Informational | No certificate-pinning implementation or Android `networkSecurityConfig` was found in the decompiled material. This is not itself a vulnerability: normal OS TLS validation still applies. | `BackendService` uses standard `HttpClient`; manifest shows no custom network security config. | Require HTTPS, keep platform trust store current, consider pinning only with an operational key-rotation plan. |

### Verify server-side with authorized test accounts

These routes take object IDs or recipient rules supplied by the client. The
app-level model does not prove that the server checks object ownership and
school membership on every operation. Verify that a low-privilege test account
cannot:

1. Read another user's chat, chat attachment, profile image, absence report,
   student record, HIP data, protocol, media attachment, or lost-property image
   by substituting a GUID in the documented route.
2. Create a chat using a manually constructed recipient rule for a person
   outside its allowed school, class, group, or relation.
3. Edit or delete another user's chat, message, absence report, lost property,
   or after-school pickup record by changing an ID in a permitted request.
4. Use a one-time token more than once, after expiry, for a different school,
   or with a changed `returnurl`.
5. Register, replace, or delete a push installation belonging to a different
   account or device.

Expected secure behavior is `401`, `403`, or `404` without revealing whether
the target record exists. Do not use real student data for these tests.

### Minimum production controls for an alternative client

* Enforce a strict Content Security Policy; do not use `unsafe-inline` scripts
  or third-party analytics on authenticated screens.
* Store tokens only in memory, or use an HttpOnly/SameSite secure server-side
  session. Do not put access tokens in URLs, localStorage, crash reports, or
  telemetry.
* Redact `Authorization`, password fields, token-like strings, message bodies,
  attachment names, and personal IDs from logs.
* Require explicit confirmation for destructive actions and show the API result
  without leaking raw server error payloads into shared screens.
* Validate uploaded size, MIME type, filename, and content server-side; scan
  files before making them available to other users.
* Apply authorization at every object route; never trust client `canEdit`,
  `isOwner`, role, or capability fields as authorization proof.

