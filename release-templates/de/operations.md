Prüfe die Release-Hinweise, setze `CRATEVIEW_VERSION` in `.env` auf den gewünschten Release-Tag mit `v`-Präfix und ersetze den Frontend-Container:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Das erste Release verwendet `CRATEVIEW_VERSION={{RELEASE_TAG}}`. Für ein Rollback stellst du den vorherigen Tag wieder ein und wiederholst die Befehle. Behalte Laufzeitkonfiguration und Volume `registry-data`; führe kein `docker compose down --volumes` aus. Quellcode-Builds bleiben über `compose.build.yaml` verfügbar.

Dependabot schlägt im Quellcode-Repository Dependency-Updates vor. Es aktualisiert dein selbst gehostetes Deployment nicht automatisch. Prüfe nach einem Upgrade Registry-Verbindung, Anmeldung und betroffene Aktionen erneut.
