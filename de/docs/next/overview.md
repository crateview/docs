---
title: 'Überblick'
description: 'Was Crateview kann, wie es deine Registry anbindet und wo seine Grenzen liegen.'
---

## Mehr Überblick über deine Registry

Crateview ist eine Weboberfläche für Docker- und OCI-Container-Registries. Du kannst Repositories und Tags finden, Multiarch-Images untersuchen, Manifeste und Konfigurationsdokumente lesen sowie komprimierte Layer und ihre Historie nachvollziehen.

Es läuft neben deiner Registry. Die Registry speichert Images und setzt Zugriffsregeln durch; Crateview liefert die Oberfläche und einen Proxy unter demselben Origin. Docker und andere OCI-Clients können den Frontend-Origin als Registry-Adresse verwenden.

![Crateview-Übersicht mit Alpine und verschachtelten Demo-Repositories.](https://crateview.de/images/docs/next/repositories-light.webp)

## Architektur

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
     Deine vorhandene OCI-Registry
```

Der Katalog benötigt keine Datenbank; optionales Monitoring nutzt persistentes SQLite. Kontodaten liegen in htpasswd und einer separaten Grants-Datei.

## Unterstützte Funktionen

- Suche nach Repository-Namen, Tag-Vorschläge und verschachtelte Namespaces.
- Tag-Tabellen, Plattformauswahl, komprimierte Layer-Größen und Image-Historie.
- OCI-Annotationen für Beschreibung, Quellcode, Dokumentation und Lizenzen.
- Manifest- und Config-JSON mit Kopierfunktion.
- OCI-Referrer einschließlich des standardisierten Digest-Tag-Index-Fallbacks, den OCI-Clients pflegen.
- Anonymer Zugriff und native htpasswd Basic Auth.
- Optionale Manifest-Löschung mit Anzeige aller betroffenen Tags.
- 13 Oberflächensprachen, System/Hell/Dunkel und anpassbares Branding.

- Monitoring ist optional und standardmäßig deaktiviert. Aktiviert öffnet Übersicht das Dashboard; sonst leitet /home auf /explore weiter. Aktivierung gibt allen Nutzern mit UI-Zugriff Registry-weite Betriebsdaten frei, bei öffentlichen Registries auch anonymen Besuchern. Verwalteter Token-Modus erfordert zusätzlich einen globalen Lese-Grant.

## Grenzen kennen

Die Standard-API liefert keine Push-Zeitpunkte, Downloadstatistiken, Favoriten oder Schwachstellen-Scans. Crateview erfindet diese Werte nicht. Das Erstellungsdatum eines Images stammt aus dessen Metadaten und ist kein Upload-Zeitpunkt. Größenangaben beschreiben komprimierte Image-Layer, nicht die Speicherbelegung der Registry.

Administration bündelt Einstellungen und Benutzer. Das Kontomenü bietet zusätzlich Sprache und Theme. Unter Konto änderst du dein eigenes Passwort mit dem aktuellen Passwort. Benutzer listet und erstellt Konten nur bei erlaubender Server-Allowlist; Token-Grants sind schreibgeschützt. Neue Passwörter benötigen mindestens 12 Zeichen und höchstens 72 UTF-8-Bytes. Neue Token-Konten besitzen zunächst keinen Repository-Zugriff.

Die Server-Allowlist steuert Auflisten, Anlegen, Passwort-Resets und Löschen von Konten. Im rechtsbündigen Drei-Punkte-Menü können Administratoren fremde Passwörter ohne das bisherige Passwort zurücksetzen und andere Konten nach Bestätigung löschen. Passwort-Resets erhalten Repository-Rechte; beim Löschen werden sie entfernt. Das eigene Menü öffnet Konto für eine Passwortänderung mit dem aktuellen Passwort; Selbstlöschung ist gesperrt.

Beginne mit der [Installation](/de/docs/next/installation) und prüfe anschließend [Konfiguration](/de/docs/next/configuration) und [Authentifizierung](/de/docs/next/authentication).

## Als App installieren (PWA)

Installierbare App mit lokalisiertem Offline-Starthinweis, Wiederholen-Aktion und konfigurierbaren Icons. Registry-Zugriff und Kontoaktionen benötigen eine Netzwerkverbindung.

[Als App installieren (PWA)](/de/docs/next/installation#pwa)
