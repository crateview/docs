---
title: 'Konfiguration'
description: 'Registry-Verbindung, Umgebungsvariablen zur Laufzeit und Verhalten beim Deployment.'
---

## Konfigurationsebenen

Die Standardwerte stehen in `nuxt.config.ts` und `app/app.config.ts`. Umgebungsvariablen überschreiben Laufzeit-Defaults beim Serverstart. Entwicklungs- und Build-Befehle lesen `.env`; der produktive Node-Einstiegspunkt tut das nicht automatisch.

Starte den Server nach Änderungen an Laufzeitvariablen neu. Änderungen an Komponenten-Konfiguration im Quellcode benötigen einen neuen Produktionsbuild.

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

Öffentliche Konfiguration wird an den Browser übertragen. Speichere niemals Passwörter oder Tokens in `NUXT_PUBLIC_*`. Farb- und Schriftoptionen stehen unter [Branding und Theming](/de/docs/v0.1/theming).

## Netzaufbau

Der Browser spricht Crateview unter einem Origin an. Nitro leitet `/v2/**` für Registry-Clients weiter und liest Registry-Metadaten über `/api/registry/**`. Ein direkter Cross-Origin-Zugriff des Browsers auf die Registry ist dadurch nicht erforderlich.

Hinter einem Reverse Proxy müssen öffentlicher Origin und Protokoll-Header erhalten bleiben. Die öffentliche Frontend-Adresse erscheint auch in kopierten Pull-Befehlen. Für authentifizierten Netzwerkzugriff muss HTTPS korrekt eingerichtet sein, bevor Zugangsdaten aus Browser oder CLI übertragen werden.

## Compose und separate Konfiguration

Die mitgelieferte Compose-Datei verbindet das Frontend bewusst über `NUXT_REGISTRY_UPSTREAM=http://registry:5000` mit ihrem eigenen `registry`-Service. Für eine andere Registry in einem Container-Deployment musst du die Service-Umgebung anpassen oder das Frontend separat starten. Eine Registry auf dem Loopback-Interface des Hosts ist nicht automatisch über ein Container-Gateway erreichbar.
