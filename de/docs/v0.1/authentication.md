---
title: 'Authentifizierung'
description: 'Anonymer Zugriff, Basic Auth, verschlüsselte Sitzungen und HTTPS.'
---

## Unterstützte Anmeldung

Anonymer Zugriff und native htpasswd-Basic-Auth werden unterstützt. Externe Bearer-Token-Dienste und OIDC werden nicht unterstützt. Benutzerverwaltung ist in diesem Release nicht verfügbar.

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
