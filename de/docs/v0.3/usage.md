---
title: 'Crateview verwenden'
description: 'Images finden, Plattformen wechseln und die Daten hinter einem Tag untersuchen.'
---

## Repositories und Tags finden

Entdecken listet Repositories. Die Suche unterstützt Repository-Namen und repository:tag. Öffne ein Repository für seine Tags.

Repositories erscheinen standardmäßig als Tabelle. Tabelle und Raster sind umschaltbar; Namen mit Schrägstrichen bilden Namespaces. Die Sidebar lässt sich vergrößern und einklappen; ihr Zustand wird lokal gespeichert. Die Suche liegt in der Sidebar.

## Ein Image untersuchen

Öffne ein Repository und wähle einen Tag. Die Image-Ansicht zeigt Digest, verfügbare Plattformen, komprimierte Layer-Größen und Erstellungsmetadaten.

Bei einem Multiarch-Image wählst du Betriebssystem und Architektur über die Plattformauswahl. Layer-Historie und ausgewählte Image-Config folgen dieser Auswahl. Der Tab Manifest zeigt das Root-Manifest oder den Index des Images; Konfiguration zeigt die Config der ausgewählten Plattform.

![Alpine-Image mit Plattformauswahl, Layer-Historie und Pull-Befehl.](https://crateview.de/images/docs/v0.3/image-light.webp)

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
