---
title: 'Konfiguration'
description: 'Registry-Verbindung, Umgebungsvariablen zur Laufzeit und Verhalten beim Deployment.'
---

## Konfigurationsebenen

Die Standardwerte stehen in `nuxt.config.ts` und `app/app.config.ts`. Umgebungsvariablen überschreiben Laufzeit-Defaults beim Serverstart. Entwicklungs- und Build-Befehle lesen `.env`; der produktive Node-Einstiegspunkt tut das nicht automatisch.

Starte den Server nach Änderungen an Laufzeitvariablen neu. Änderungen an Komponenten-Konfiguration im Quellcode benötigen einen neuen Produktionsbuild.

## Datums- und Uhrzeitformat

Setze `NUXT_PUBLIC_DATE_FORMAT` zur Laufzeit, um alle Datumsanzeigen zu formatieren. Die Zeitzone kommt vom Browser. Ist die Variable leer oder ungültig, verwendet Crateview das Browserformat.

Beispiel: `DD.MM.YYYY HH:mm`. Platzhalter: `YYYY`, `MM`, `DD`, `HH`, `mm`, `ss`, `Z` (UTC ±HH:mm). Text in eckige Klammern setzen, etwa `[um]`.

## Registry- und Sitzungseinstellungen

| Variable                       | Standard                | Zweck                                           |
| ------------------------------ | ----------------------- | ----------------------------------------------- |
| `NUXT_REGISTRY_UPSTREAM`       | `http://127.0.0.1:5000` | Serverseitiger Registry-Origin.                 |
| `NUXT_REGISTRY_DELETE_ENABLED` | `false`                 | Manifest-Löschung im UI anbieten.               |
| `NUXT_SESSION_PASSWORD`        | Zufällig pro Prozess    | Schlüssel zur Verschlüsselung der UI-Sitzungen. |
| `HOST`                         | Laufzeitabhängig        | Interface des Node-Servers.                     |
| `PORT`                         | `3000`                  | Port des Node-Servers.                          |

Der Upstream muss ein vom Frontend-Server erreichbarer HTTP(S)-Origin sein. Zugangsdaten gehören nicht in die URL. Anfragen verwenden die Authentifizierung des aktuellen Benutzers; Crateview ersetzt die Zugriffsregeln der Registry nicht.

## Öffentliche Branding-Variablen

Alle folgenden Namen verwenden das Präfix `NUXT_PUBLIC_BRAND_`:

| Suffix                                      | Zweck                                                     |
| ------------------------------------------- | --------------------------------------------------------- |
| `NAME`                                      | Name der Anwendung.                                       |
| `DESCRIPTION`                               | Beschreibung der Anwendung.                               |
| `LOGO_LIGHT`, `LOGO_DARK`                   | Logo-Pfade oder HTTP(S)-URLs.                             |
| `FAVICON`                                   | Pfad oder URL des Browser-Icons.                          |
| `TITLE_TEMPLATE`                            | Seitentitel; `%s` wird durch den Titel der Seite ersetzt. |
| `PRODUCT_URL`, `DOCS_URL`, `REPOSITORY_URL` | Optionale Navigationslinks.                               |

Öffentliche Konfiguration wird an den Browser übertragen. Speichere niemals Passwörter oder Tokens in `NUXT_PUBLIC_*`. Farb- und Schriftoptionen stehen unter [Branding und Theming](/de/docs/next/theming).

## Netzaufbau

Der Browser spricht Crateview unter einem Origin an. Nitro leitet `/v2/**` für Registry-Clients weiter und liest Registry-Metadaten über `/api/registry/**`. Ein direkter Cross-Origin-Zugriff des Browsers auf die Registry ist dadurch nicht erforderlich.

Hinter einem Reverse Proxy müssen öffentlicher Origin und Protokoll-Header erhalten bleiben. Die öffentliche Frontend-Adresse erscheint auch in kopierten Pull-Befehlen. Für authentifizierten Netzwerkzugriff muss HTTPS korrekt eingerichtet sein, bevor Zugangsdaten aus Browser oder CLI übertragen werden.

## Compose und separate Konfiguration

Die mitgelieferte Compose-Datei verbindet das Frontend bewusst über `NUXT_REGISTRY_UPSTREAM=http://registry:5000` mit ihrem eigenen `registry`-Service. Für eine andere Registry in einem Container-Deployment musst du die Service-Umgebung anpassen oder das Frontend separat starten. Eine Registry auf dem Loopback-Interface des Hosts ist nicht automatisch über ein Container-Gateway erreichbar.
## Versionsspezifische Konfiguration

Monitoring ist optional und standardmäßig deaktiviert. Aktiviert öffnet Übersicht das Dashboard; sonst leitet /home auf /explore weiter. Aktivierung gibt allen Nutzern mit UI-Zugriff Registry-weite Betriebsdaten frei, bei öffentlichen Registries auch anonymen Besuchern. Verwalteter Token-Modus erfordert zusätzlich einen globalen Lese-Grant.

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

[Monitoring](/de/docs/next/monitoring)

| ENV | Default |
| --- | --- |
| `NUXT_AUTH_MODE` | `off` |
| `NUXT_AUTH_SERVICE_URL` | — |
| `NUXT_AUTH_SERVICE_SECRET` | — |
| `NUXT_AUTH_TOKEN_REALM` | — |
| `NUXT_USER_CREATION_ALLOWED_USERS` | — |

Administration bündelt Einstellungen und Benutzer. Das Kontomenü bietet zusätzlich Sprache und Theme. Unter Konto änderst du dein eigenes Passwort mit dem aktuellen Passwort. Benutzer listet und erstellt Konten nur bei erlaubender Server-Allowlist; Token-Grants sind schreibgeschützt. Neue Passwörter benötigen mindestens 12 Zeichen und höchstens 72 UTF-8-Bytes. Neue Token-Konten besitzen zunächst keinen Repository-Zugriff.

Die Server-Allowlist steuert Auflisten, Anlegen, Passwort-Resets und Löschen von Konten. Im rechtsbündigen Drei-Punkte-Menü können Administratoren fremde Passwörter ohne das bisherige Passwort zurücksetzen und andere Konten nach Bestätigung löschen. Passwort-Resets erhalten Repository-Rechte; beim Löschen werden sie entfernt. Das eigene Menü öffnet Konto für eine Passwortänderung mit dem aktuellen Passwort; Selbstlöschung ist gesperrt.

## PWA-Icons konfigurieren

| Variable | Standard | Zweck |
| --- | --- | --- |
| `NUXT_PUBLIC_BRAND_PWA_ICON_192` | `/branding/pwa-192.png` | URL des Installationsicons mit 192 Pixeln. |
| `NUXT_PUBLIC_BRAND_PWA_ICON_512` | `/branding/pwa-512.png` | URL des Installationsicons mit 512 Pixeln. |
| `NUXT_BRAND_PWA_ICON_192_FILE` | — | Private PNG-Datei mit 192 Pixeln als Override. |
| `NUXT_BRAND_PWA_ICON_512_FILE` | — | Private PNG-Datei mit 512 Pixeln als Override. |

Lokale Icon-Dateien haben Vorrang vor den jeweiligen öffentlichen URLs. Erforderlich sind lesbare, nicht leere PNGs mit exakt den quadratischen Abmessungen. Ungültige Dateien verhindern den Serverstart. Relative Dateipfade beziehen sich auf das Arbeitsverzeichnis des Servers; Containerpfade müssen auf lesbare Dateien im Container zeigen. Nach Konfigurationsänderungen neu starten.

Lokale Icon-URLs und das Manifest berücksichtigen `NUXT_APP_BASE_URL`, beispielsweise `/registry/`; externe HTTP(S)-Icon-URLs bleiben erhalten. Die privaten Dateivariablen sind kein Geheimnisspeicher: Die Bilddaten sind öffentlich, nur die Dateisystempfade bleiben serverseitig.
