# Realtime und Push

## SignalR

| Element | Beschreibung |
|---|---|
| Hub | `{origin}/notifyHub` |
| Authentifizierung | Bearer-Token wird als SignalR-Access-Token bereitgestellt. |
| Reconnect | Der Client aktiviert automatische Wiederverbindung. |
| Event | `newmessage(NotifyHubNewMessage)` |
| Sichtbare Nutzlast | Mindestens `chatId` und Nachrichten-`id`. |
| Clientreaktion | Die App aktualisiert den zugehörigen Chat. |

Die vollständigen Event-Payloads, weitere Hub-Methoden und Verbindungsfehler
sind in der Spezifikation nicht beschrieben.

## Push-Installationen

| Methode | Route (relativ zu `/Api/v1.0`) | Request |
|---|---|---|
| PUT | `/App/PushNotifications/Installations` | JSON-Modell `DeviceInstallation`. |
| DELETE | `/App/PushNotifications/Installations` | JSON-Modell `DeviceInstallation`. |

Das native `DeviceInstallation`-Modell enthält Plattform-Push-Kanal,
Installationskennung und Tags. Die App verwendet gerätelokale Push-Tokens.
Ein Webclient sollte stattdessen die Browser Push API und einen eigenen
Service Worker verwenden und keine mobilen Installationsdaten kopieren.

## Schutz

Push-Tokens und Installationskennungen sind vertrauliche Gerätekennungen.
Sie nicht protokollieren oder zwischen Benutzern teilen. Der Server muss
Registrierung, Änderung und Löschung einer Installation dem berechtigten
Konto/Gerät zuordnen.
