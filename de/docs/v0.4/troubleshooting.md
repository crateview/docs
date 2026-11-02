---
title: 'Fehlerbehebung'
description: 'Probleme mit Verbindung, Anmeldung, Metadaten und Deployment eingrenzen.'
---

## Das UI erreicht die Registry nicht

Prüfe den Registry-Origin aus der Netzwerkumgebung des Frontend-Servers. In einem Container zeigt `localhost` auf den Container selbst. Ein üblicher Compose-Upstream lautet `http://registry:5000`.

```sh
curl -i http://127.0.0.1:5000/v2/
```

Eine anonyme Registry sollte Erfolg melden. Eine Registry mit Anmeldung kann `401` und einen `WWW-Authenticate`-Header liefern. Bei einer abgelehnten Verbindung sind Listener, Adresse oder Netzwerkpfad nicht bereit.

## Die Anmeldung schlägt fehl

Prüfe, ob die Registry native htpasswd Basic Auth verwendet und ihre bcrypt-Datei korrekt eingebunden ist. Eine externe Bearer-Token-Anforderung unterstützt das aktuelle UI nicht.

Bei wiederholtem Sitzungsverlust müssen alle Frontend-Instanzen denselben `NUXT_SESSION_PASSWORD` verwenden. Ein automatisch erzeugter prozesslokaler Schlüssel ändert sich beim Neustart. Prüfe außerdem HTTPS und weitergeleitete Protokoll-Header; der Browser muss das Sitzungscookie an denselben Origin zurücksenden können.

## Beschreibung oder Erstellungszeit fehlen

Diese Werte müssen in Manifest-Annotationen oder Image-Config vorhanden sein. Crateview leitet fehlende Metadaten nicht ab und setzt keine Push-Zeitpunkte als Ersatz ein. Ergänze die passenden OCI-Annotationen beim Image-Build.

## Referenzen bleiben leer

Die Registry muss die OCI-Referrers-API oder den standardisierten Digest-Tag-Fallback-Index bereitstellen. OCI-Clients pflegen diesen Index, falls er benötigt wird. Ein Manifest mit Subject-Feld allein veranlasst Crateview nicht dazu, bei einer Registry ohne Referrers-Unterstützung alle Tags zu durchsuchen.

## Die Löschfunktion fehlt oder wird abgelehnt

Aktiviere sowohl `NUXT_REGISTRY_DELETE_ENABLED` in Crateview als auch die Storage-Delete-Einstellung in Distribution. Die Registry kann Anfragen weiterhin anhand ihrer Berechtigungen oder Implementierung ablehnen. Gelöscht wird per Digest; mehrere Tags können betroffen sein.

## Der Produktivbetrieb ignoriert .env

`node .output/server/index.mjs` lädt `.env` nicht automatisch. Übergib die Variablen über Prozessmanager, Container-Umgebung oder Shell. Starte nach Änderungen an Laufzeitwerten neu. Ein rein statischer Webhost ersetzt den benötigten Nitro-Server nicht.

## Der kopierte Pull-Befehl funktioniert nicht

Der Befehl verwendet die öffentliche Frontend-Adresse. Prüfe, ob die CLI diesen Origin erreicht und seinem TLS-Zertifikat vertraut. Melde die CLI gegebenenfalls separat an. Eine Browser-Anmeldung erzeugt keine Docker-CLI-Anmeldung.

Melde reproduzierbare UI-Fehler als [Issue](https://github.com/crateview/registry-ui/issues) mit betroffener Version und bereinigten Reproduktionsschritten. Für Sicherheitslücken nutze den [privaten Meldeweg](https://github.com/crateview/registry-ui/security/policy).
## Monitoring

Versionsspezifische Konfiguration: [Monitoring](/de/docs/v0.4/monitoring).

Versionsspezifische Konfiguration — Monitoring ist optional und standardmäßig deaktiviert. Aktiviert öffnet Übersicht das Dashboard; sonst leitet /home auf /explore weiter. Aktivierung gibt allen Nutzern mit UI-Zugriff Registry-weite Betriebsdaten frei, bei öffentlichen Registries auch anonymen Besuchern.
