---
title: 'Crateview verwenden'
description: 'Images finden, Plattformen wechseln und die Daten hinter einem Tag untersuchen.'
---

## Repositories und Tags finden

Entdecken listet Repositories. Die Suche unterstützt Repository-Namen und repository:tag. Öffne ein Repository für seine Tags.

Repositories erscheinen standardmäßig als Tabelle. Tabelle und Raster sind umschaltbar; Namen mit Schrägstrichen bilden Namespaces. Die Sidebar lässt sich vergrößern und einklappen; ihr Zustand wird lokal gespeichert. Die Suche liegt in der Sidebar.

/ oder Ctrl+K (Cmd+K unter macOS) öffnet die Befehlspalette für Repository-/Tag-Suche und Navigation. Namespace-Ordner sind einklappbar. Einstellungen enthält Sprache, System/Hell/Dunkel und Über mit durchsuchbaren Abhängigkeitslizenzen und Originalhinweisen.

Administration bündelt Einstellungen und Benutzer. Das Kontomenü bietet zusätzlich Sprache und Theme. Unter Konto änderst du dein eigenes Passwort mit dem aktuellen Passwort. Benutzer listet und erstellt Konten nur bei erlaubender Server-Allowlist; Token-Grants sind schreibgeschützt. Neue Passwörter benötigen mindestens 12 Zeichen und höchstens 72 UTF-8-Bytes. Neue Token-Konten besitzen zunächst keinen Repository-Zugriff.

Die Server-Allowlist steuert Auflisten, Anlegen, Passwort-Resets und Löschen von Konten. Im rechtsbündigen Drei-Punkte-Menü können Administratoren fremde Passwörter ohne das bisherige Passwort zurücksetzen und andere Konten nach Bestätigung löschen. Passwort-Resets erhalten Repository-Rechte; beim Löschen werden sie entfernt. Das eigene Menü öffnet Konto für eine Passwortänderung mit dem aktuellen Passwort; Selbstlöschung ist gesperrt.

![Branding & Theming](https://crateview.de/images/docs/v0.7/settings-light.webp)

## Ein Image untersuchen

Öffne ein Repository und wähle einen Tag. Die Image-Ansicht zeigt Digest, verfügbare Plattformen, komprimierte Layer-Größen und Erstellungsmetadaten.

Bei einem Multiarch-Image wählst du Betriebssystem und Architektur über die Plattformauswahl. Layer-Historie und ausgewählte Image-Config folgen dieser Auswahl. Der Tab Manifest zeigt das Root-Manifest oder den Index des Images; Konfiguration zeigt die Config der ausgewählten Plattform.

![Alpine-Image mit Plattformauswahl, Layer-Historie und Pull-Befehl.](https://crateview.de/images/docs/v0.7/image-light.webp)

## Pull-Befehle

Das Befehlspanel bietet Docker-, Podman- und Skopeo-Befehle für Images sowie ORAS-/Podman-Artefaktbefehle für Artefakte. Kopiere den Befehl und führe ihn in deinem Terminal aus.

```sh
docker pull registry.example.com/team/service:latest
podman pull registry.example.com/team/service:latest
oras pull registry.example.com/team/report:latest
```

Ersetze den Beispielhost durch den öffentlichen Crateview-Origin. CLI-Anmeldung und Browser-Sitzung sind voneinander getrennt.

## OCI-Metadaten und Referrer

Beschreibungen, Lizenzen und Quellcode-Links werden aus OCI-Annotationen oder Image-Config-Labels gelesen, sofern sie vorhanden sind. Fehlende Metadaten werden nicht geraten.

Der Tab Referenzen liest die OCI-Referrers-API. Bietet die Registry diesen Endpunkt nicht an, liest Crateview den standardisierten Digest-Tag-Index, den OCI-Clients pflegen. Es erstellt keinen eigenen Index durch das Durchsuchen aller Manifeste. Aufgeführte Signatur-Artefakte werden nicht automatisch geprüft oder als vertrauenswürdig markiert.

## Zeitpunkte und Größen verstehen

Die Erstellungszeit stammt aus Image-Config oder Annotationen. Sie beschreibt nicht, wann ein Tag in diese Registry gepusht wurde. Komprimierte Layer-Größen messen keine Speicherbelegung: Layer können gemeinsam genutzt werden und gelöschte Daten bis zur Garbage Collection erhalten bleiben.
