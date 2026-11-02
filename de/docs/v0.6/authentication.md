---
title: 'Authentifizierung'
description: 'Anonymer Zugriff, Basic Auth, verschlüsselte Sitzungen und HTTPS.'
---

## Unterstützte Anmeldung

Crateview unterstützt anonyme Registries, native htpasswd Basic Auth und den optionalen Crateview-Auth-Dienst im Token-Modus. Es prüft `/v2/` und zeigt das Anmeldeformular, wenn die Registry Basic-Authentifizierung verlangt.

Externe Bearer-Token-Dienste und OIDC sind nicht integriert. Nicht konfigurierte externe Token-Dienste führen zu einem ausdrücklichen Hinweis auf die nicht unterstützte Anmeldung.

## Basic Auth auf Registry-Seite

Erstelle eine htpasswd-Datei mit bcrypt-Passwort-Hash. Das folgende Beispiel verwendet einen temporären offiziellen Apache-Container. Ersetze die Beispielzugangsdaten und schütze die erzeugte Datei:

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Beispiel für Distribution:

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

Binde die Datei im Registry-Container unter `/auth/htpasswd` ein. Die Registry wird getrennt von Crateview konfiguriert. Verwende HTTPS für Netzwerkzugriff; Basic Auth kodiert Zugangsdaten, verschlüsselt aber nicht den Transport. Echte Passwörter gehören weder in gemeinsam genutzte Shell-Verläufe noch in öffentliche Beispiele.

## UI-Sitzungen

Der Nitro-Server prüft die Zugangsdaten gegen die Registry. Er speichert sie in einem verschlüsselten HttpOnly-Cookie mit SameSite=Strict. Browser-JavaScript und localStorage erhalten die Zugangsdaten nicht. Sitzungen laufen nach acht Stunden ab.

Erzeuge einen zufälligen Schlüssel für den Produktivbetrieb:

```sh
openssl rand -base64 32
```

Trage den erzeugten Wert mit mindestens 32 Zeichen als `NUXT_SESSION_PASSWORD` ein. Alle Frontend-Instanzen hinter einem Load Balancer benötigen denselben Wert. Ohne Konfiguration erzeugt Crateview einen prozesslokalen Schlüssel; ein Neustart macht bestehende Sitzungen ungültig.

## HTTPS und Abmeldung

Bei HTTPS als öffentlichem Anfrageprotokoll erhält das Sitzungscookie das Secure-Attribut. Konfiguriere das weitergeleitete Protokoll am Reverse Proxy korrekt. Die Abmeldung löscht die UI-Sitzung, verändert aber keine Registry-Benutzer.

Docker und andere CLI-Clients melden sich über den `/v2/**`-Proxy mit ihren normalen Registry-Zugangsdaten an. Eine Anmeldung im Web-UI meldet die Docker-CLI nicht automatisch an.

## Optionale Benutzerverwaltung

Ein separater Auth-/Verwaltungsdienst ermöglicht Benutzeranlage und eigene Passwortänderungen. Docker verbindet sich direkt mit der Registry und funktioniert unabhängig vom UI-Container. Benutzer und bcrypt-Hashes bleiben in `htpasswd`; Token-Rechte stehen in `permissions.json`. Keine SQLite-Speicherung für Konten.

- **Basic-Modus:** Distribution prüft die gemeinsame htpasswd-Datei; alle Benutzer haben dieselben Registry-Rechte. Der Verwaltungsdienst muss für Docker-Zugriffe nicht laufen.
- **Token-Modus:** Der eigenständige Auth-Dienst stellt fünf Minuten gültige signierte Tokens aus; Distribution erzwingt Repository-Rechte. Docker benötigt Registry und Auth-Dienst. Neue Benutzer haben zunächst keine Repository-Rechte.
- **UI:** Die Server-Allowlist steuert Auflisten, Anlegen, Passwort-Resets und Löschen von Konten. `NUXT_USER_CREATION_ALLOWED_USERS`: `alice,bob`; `*`. Leer verweigert die Verwaltung; aufgeführte Namen sind erlaubt, der Platzhalter erlaubt alle angemeldeten Benutzer.
- **Rechte:** CLI-Zuweisungen für exakte Repositories, Namespaces (`team/*`) oder global (`*`); Profile `reader` (Pull), `writer` (Pull/Push), `manager` (Pull/Push/Löschen). Im Basic-Modus gibt es keine Registry-Rollen.

Aktiviere explizit `NUXT_AUTH_MODE=htpasswd` oder `token`, setze die interne `NUXT_AUTH_SERVICE_URL` und das serverseitige `NUXT_AUTH_SERVICE_SECRET`. Im Token-Modus muss `NUXT_AUTH_TOKEN_REALM` genau dem öffentlichen Token-Endpunkt entsprechen. Anonyme Registries und fremde Token-Dienste bieten keine lokale Benutzerverwaltung. Repository-Rechte werden per Auth-CLI vergeben; die UI zeigt sie schreibgeschützt an.

Compose-Dateien, Einrichtung, Migration und CLI-Befehle stehen im [Auth-Dienst-Handbuch](https://github.com/crateview/registry-ui/blob/8fe2c00e9419faf971c2ab66fa3a561cb3dc6c00/auth/README.md). Beide Auth-Modi sind optionale Compose-Overlays; der öffentliche Docker-Zugang muss unabhängig zum Registry-Dienst geroutet werden. Bereits ausgestellte Tokens bleiben nach Passwortänderungen oder Rechteentzug bis zu fünf Minuten gültig.

Administration bündelt Einstellungen und Benutzer. Das Kontomenü bietet zusätzlich Sprache und Theme. Unter Konto änderst du dein eigenes Passwort mit dem aktuellen Passwort. Benutzer listet und erstellt Konten nur bei erlaubender Server-Allowlist; Token-Grants sind schreibgeschützt. Neue Passwörter benötigen mindestens 12 Zeichen und höchstens 72 UTF-8-Bytes. Neue Token-Konten besitzen zunächst keinen Repository-Zugriff.

Die Server-Allowlist steuert Auflisten, Anlegen, Passwort-Resets und Löschen von Konten. Im rechtsbündigen Drei-Punkte-Menü können Administratoren fremde Passwörter ohne das bisherige Passwort zurücksetzen und andere Konten nach Bestätigung löschen. Passwort-Resets erhalten Repository-Rechte; beim Löschen werden sie entfernt. Das eigene Menü öffnet Konto für eine Passwortänderung mit dem aktuellen Passwort; Selbstlöschung ist gesperrt.

![Dokumentationsversion](https://crateview.de/images/docs/v0.6/users-light.webp)

![Aktionen](https://crateview.de/images/docs/v0.6/user-actions-light.webp)

![Passwort ändern](https://crateview.de/images/docs/v0.6/user-password-light.webp)

![Benutzer löschen](https://crateview.de/images/docs/v0.6/user-delete-light.webp)

```dotenv
NUXT_AUTH_MODE=htpasswd
NUXT_AUTH_SERVICE_URL=http://auth:8080
NUXT_AUTH_SERVICE_SECRET=REPLACE_WITH_SEPARATE_MANAGEMENT_SECRET
NUXT_USER_CREATION_ALLOWED_USERS=alice,bob
```

![Dokumentationsversion](https://crateview.de/images/docs/v0.6/account-light.webp)

![Dokumentationsversion](https://crateview.de/images/docs/v0.6/login-light.webp)
