---
title: 'Betrieb & Löschen'
description: 'Manifeste bewusst verwalten und Registry-Daten unter eigener Kontrolle halten.'
---

## Manifest-Löschung aktivieren

Die Löschfunktion ist im Crateview-UI standardmäßig deaktiviert. Sowohl das Frontend als auch Distribution müssen sie erlauben:

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution kann auch `REGISTRY_STORAGE_DELETE_ENABLED=true` lesen. Starte die betreffenden Dienste nach Änderungen an Umgebung oder Konfiguration neu. Die Registry bleibt für die Autorisierung zuständig, auch bei direkten Anfragen über `/v2/**`.

## Digest-Löschung verstehen

Die standardisierte Löschoperation verwendet einen **Manifest-Digest**. Alle darauf zeigenden Tags können entfernt werden. Crateview zeigt die betroffenen Tags vor der Bestätigung an. Ein paralleler Schreibzugriff kann Referenzen danach aber ändern: Die Registry-API bietet keine Transaktion zum Sperren der Aliasse.

Betrachte die Aktion als Änderung am Manifest und allen seinen Alias-Tags, nicht als Entfernen eines einzelnen Labels. Prüfe Backups und beabsichtigten Umfang vor der Bestätigung.

## Garbage Collection und Backups

Das Löschen eines Manifests gibt nicht zwangsläufig sofort Speicher frei. Garbage Collection ist eine separate Registry-Verwaltungsaufgabe; Crateview führt sie nicht aus.

Sichere Registry-Daten und Konfiguration mit einem zum Storage-Treiber passenden Verfahren. Behalte Daten-Volumes beim Austausch von Containern bei. Beachte die Distribution-Anleitung für Garbage Collection und die Koordination von Schreibzugriffen:

- [Distribution Garbage Collection](https://distribution.github.io/distribution/about/garbage-collection/)
- [Distribution-Konfiguration](https://distribution.github.io/distribution/about/configuration/)

## Crateview aktualisieren

Prüfe die Release-Hinweise, setze `CRATEVIEW_VERSION` in `.env` auf den gewünschten Release-Tag mit `v`-Präfix und ersetze den Frontend-Container:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Das erste Release verwendet `CRATEVIEW_VERSION=v0.2.1`. Für ein Rollback stellst du den vorherigen Tag wieder ein und wiederholst die Befehle. Behalte Laufzeitkonfiguration und Volume `registry-data`; führe kein `docker compose down --volumes` aus. Quellcode-Builds bleiben über `compose.build.yaml` verfügbar.

Dependabot schlägt im Quellcode-Repository Dependency-Updates vor. Es aktualisiert dein selbst gehostetes Deployment nicht automatisch. Prüfe nach einem Upgrade Registry-Verbindung, Anmeldung und betroffene Aktionen erneut.

## Große Registries

Die Repository-Suche liest den vollständigen paginierten Katalog. Metadaten sichtbarer Repositories werden bei Bedarf geladen. Die Sortierung einer großen Tag-Liste nach Image-Erstellung kann viele Manifest- und Config-Anfragen erfordern. Crateview betreibt keinen Hintergrund-Metadatenindex oder externen Suchdienst.
