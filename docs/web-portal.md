# Webportal, Home.InfoPoint und Stundenplan

Dieser Abschnitt dokumentiert eine separate, cookie-authentifizierte
Weboberfläche. Die Routen und Parsing-Felder stammen aus dem verlinkten
Drittprojekt und wurden nicht für jede Schulinstanz bestätigt. Ein mobiler
Bearer-JWT ersetzt den Portal-Cookie nicht.

## Portal-Login und Seiten

| Methode | Route | Request / Voraussetzung | Verwendung |
|---|---|---|---|
| GET | `/account/login` | Cookie-Jar aktiviert | HTML-Loginformular mit verborgenem `__RequestVerificationToken`. |
| POST | `/account/login` | Formfelder `UserName`, `Password`, `RememberMe=false`, `__RequestVerificationToken` | Bei Erfolg Portal-Session-Cookie. Ein erneut gerendertes Passwortformular ist als fehlgeschlagener Login zu behandeln. |
| GET | `/{schoolSlug}/hip` | Authentifizierter Portal-Cookie | Gerendertes Home.InfoPoint-HTML. |
| GET | `/{schoolSlug}/schedule` | Authentifizierter Portal-Cookie | Stundenplanseite; ausgewählter Schüler ist eine Option in `#Selection` mit Wert `s:{studentId}`. |
| GET | `/{schoolSlug}/schedule?handler=Schedule&date={YYYY-MM-DD}&cog=&t=&s={studentId}&view=` | Authentifizierter Portal-Cookie | JSON für die Stundenplanwoche mit `date`. |

Cookie-Jar und Origin müssen erhalten bleiben. Redirects dürfen nur innerhalb
derselben Origin verfolgt werden. Eine Weiterleitung nach
`/account/accessdenied` oder eine zurückgelieferte Loginseite bedeutet
Autorisierungsfehler.

## Home.InfoPoint-HTML

Der Referenzparser erwartet `.main.withschoolmenu`, einen `h2`-Schülernamen
und deutsche Abschnittsüberschriften. Die dokumentierte Zuordnung:

| Überschrift | Zeilenstruktur | Felder |
|---|---|---|
| `Noten` | Jedes Fach ist ein `h4`; nachfolgende Tabellenzeilen mit vier Zellen | `date`, `value`, `remark`, `information` |
| `Fehlzeiten` | Tabellenzeilen mit zwei Zellen | `date`, `remark` |
| `Bemerkungen` | Tabellenzeilen mit fünf Zellen | `date`, `text`, `subject`, `type`, `author` |
| `Unterricht` | Tabellenzeilen mit drei Zellen | `date`, `subject`, `homework` |

Datumsangaben werden als `DD.MM.YYYY` angezeigt. Die Formulierung
`zum DD.MM.YYYY` in einem Hausaufgabentext ist eine explizite, im Text
enthaltene Frist und kein separates Serverfeld. Eine anderweitige Fristableitung
aus der nächsten passenden, nicht abgesagten Unterrichtsstunde ist Clientlogik
und muss als solche gekennzeichnet werden.

## Stundenplan-JSON (bestätigt)

Die Struktur wurde an echten Antworten bestätigt. Details (Felder `startTimes`,
`infos`, `schedule.days[].l[]`, Eintragsfelder `st/ct/gt/tt/rm/txt/rix/cancelled`,
`replacements[]`-Zeilen) stehen im englischen Abschnitt „Timetable JSON structure“ in
[MSD-API-SPEC.md](../MSD-API-SPEC.md). Kurzfassung:

* `cog` erwartet den **Namen** der Klasse/Gruppe (`10c`, `En1_10_8b`), keine ID – mit ID liefert der Server `schedule: null`.
* `view=` leer liefert die Vertretungsübersicht (`replacements`, `infos`).
* Ein Eintrag mit `rix >= 0` verweist auf `replacements[rix]` derselben Antwort (alt → neu für Fach, Lehrer, Raum; Verlegung über `newDate`/`newLessonIndex`).
* `startTimes` enthält nur Startzeiten (`"08:00:00"`), keine Endzeiten.
* `infos[].informationPublic` sind Tageshinweise wie „Präsenz: Lie“.

## Sicherer Umgang

Portal-Cookies gehören in einen HttpOnly-/Secure-/SameSite-geschützten
Speicher, nicht in für JavaScript zugänglichen Storage. Formular-Action und
Redirect-Origin prüfen, CSRF-Token verwenden und keine Portal-Cookies in
Logs oder Diagnoseausgaben schreiben. Weitere Hinweise enthält die
[Sicherheitsbetrachtung](security.md).
