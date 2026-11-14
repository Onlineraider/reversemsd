# Authentifizierung und Konto

Die mobilen API-Aufrufe verwenden Bearer-Token. Die Webportal-Anmeldung ist
davon getrennt und verwendet Cookies sowie einen Antiforgery-Token. Die beiden
Anmeldeverfahren sind nicht austauschbar.

## Token- und Profilrouten

| Methode | Route (relativ zu `/Api/v1.0`) | Request | Beobachtete Antwort / Verwendung |
|---|---|---|---|
| POST | `/Token/CreateToken` | `{ userName, password, twoFactorCode: string|null, appId }` | `TokenResultModel`; Erfolg enthält `token`, `refreshToken`, `expireIn`. Eine 401-Antwort kann `twoFactorNeeded` enthalten. |
| POST | `/Token/RefreshToken` | `{ token, refreshToken, appId }` | Erneuertes Token-Paar. Die App erneuert ungefähr zwei Minuten vor Ablauf des Access-Tokens. |
| GET | `/App/App/Profile` | Bearer-Token | `Profile { userId, displayName, schools[] }`. |
| POST | `/App/App/OneTimeToken` | leerer Body, Bearer-Token | `OneTimeToken { token }`; Brücke zur Weboberfläche. |
| POST | `/Token/MSDAppExternalLogin/{scheme}/{appId}` | Browser-Authentifizierungs-/Redirect-Ablauf | Externer Login; der App-Callback verwendet `msdapp://`. |
| POST | `/Token/MSDAppExternalAppleLogin/{appId}` | `{ idToken }` | Apple-Plattform-Login. |

`ProfileSchoolInformation` enthält mindestens `name` und `tag`. Die native App
speichert den ausgewählten Schul-Tag lokal als `Auth.School`; bei einem
Schulwechsel lädt sie den Schulstatus erneut.

## Bilder und Web-Brücke

| Methode | Route | Authentifizierung | Verwendung |
|---|---|---|---|
| GET | `/Api/v1.0/Account/Image/{userId}?w={pixels}` | Bearer-Token | Benutzeravatar. Global, nicht schulbezogen. Die App fragt Breite 48 an; andere unterstützte Größen sind nicht dokumentiert. |
| GET | `/account/forgotpassword` | Browserseite | Seite zur Passwortwiederherstellung. |
| GET | `/account/delete` | Browserseite | Kontolöschungsseite. |
| GET | `/Account/Login?token={oneTimeToken}&inapp=1&returnurl=/{schoolTag}/{path}` | Browserseite | Web-UI-Brücke, u. a. für Profilbearbeitung, Stundenplan, Medien-/Kalenderpflege und Schüler-Eigenschaften. |

Für Web-UI-Routen fordert die App zuerst ein One-Time-Token an. Die
Spezifikation belegt keinen JSON-Schreibendpunkt für die genannten
Bearbeitungsseiten. Das Token ist sensibel und sollte unmittelbar vor dem
Öffnen der Seite abgerufen und nicht protokolliert werden.

## Sicherheits- und Implementierungshinweise

- Access- und Refresh-Tokens niemals im Klartext protokollieren oder dauerhaft
  im Browser speichern.
- Fehler bei Login und Refresh explizit behandeln; `twoFactorNeeded` ist kein
  erfolgreicher Login.
- Eine externe Login-Rückleitung oder `returnurl` nicht ungeprüft übernehmen.
- Details zu Cookie- und CSRF-Schutz im [Webportal-Kapitel](web-portal.md);
  allgemeine Schutzmaßnahmen stehen in der [Sicherheitsbetrachtung](security.md).
