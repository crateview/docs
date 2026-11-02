---
title: 'Überblick'
description: 'Was Crateview kann, wie es deine Registry anbindet und wo seine Grenzen liegen.'
---

## Mehr Überblick über deine Registry

Crateview ist eine Weboberfläche für Docker- und OCI-Container-Registries. Du kannst Repositories und Tags finden, Multiarch-Images untersuchen, Manifeste und Konfigurationsdokumente lesen sowie komprimierte Layer und ihre Historie nachvollziehen.

Es läuft neben deiner Registry. Die Registry speichert Images und setzt Zugriffsregeln durch; Crateview liefert die Oberfläche und einen Proxy unter demselben Origin. Docker und andere OCI-Clients können den Frontend-Origin als Registry-Adresse verwenden.

![Crateview-Übersicht mit Alpine und verschachtelten Demo-Repositories.](https://crateview.de/images/repositories-light.webp)

## Architektur

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
     Deine vorhandene OCI-Registry
```

Das Frontend verwendet die standardisierte Registry HTTP API V2. Metadaten für das UI entstehen aus Katalog, Tag-Listen, Manifesten und Image-Configs. Ein eigener Suchindex oder eine zusätzliche Anwendungsdatenbank ist nicht erforderlich.

## Unterstützte Funktionen

- Suche nach Repository-Namen, Tag-Vorschläge und verschachtelte Namespaces.
- Tag-Tabellen, Plattformauswahl, komprimierte Layer-Größen und Image-Historie.
- OCI-Annotationen für Beschreibung, Quellcode, Dokumentation und Lizenzen.
- Manifest- und Config-JSON mit Kopierfunktion.
- OCI-Referrer einschließlich des standardisierten Digest-Tag-Index-Fallbacks, den OCI-Clients pflegen.
- Anonymer Zugriff und native htpasswd Basic Auth.
- Optionale Manifest-Löschung mit Anzeige aller betroffenen Tags.
- 13 Oberflächensprachen, System/Hell/Dunkel und anpassbares Branding.

## Grenzen kennen

Die Standard-API liefert keine Push-Zeitpunkte, Downloadstatistiken, Favoriten oder Schwachstellen-Scans. Crateview erfindet diese Werte nicht. Das Erstellungsdatum eines Images stammt aus dessen Metadaten und ist kein Upload-Zeitpunkt. Größenangaben beschreiben komprimierte Image-Layer, nicht die Speicherbelegung der Registry.

Externe Bearer-Token-Dienste, OIDC, Benutzerverwaltung und API-Schlüsselverwaltung sind nicht integriert. Signatur-Artefakte können als Referrer erscheinen; das ist keine kryptografische Signaturprüfung.

Beginne mit der [Installation](/de/docs/installation) und prüfe anschließend [Konfiguration](/de/docs/configuration) und [Authentifizierung](/de/docs/authentication).
