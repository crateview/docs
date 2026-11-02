Consultez les notes de version, définissez `CRATEVIEW_VERSION` sur la balise de version préfixée `v` souhaitée dans `.env` et remplacez le conteneur frontal :

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Par exemple, la version initiale utilise `CRATEVIEW_VERSION={{RELEASE_TAG}}`. Pour revenir en arrière, restaurez la balise précédente et répétez ces commandes. Conservez l'environnement d'exécution et le volume `registry-data` ; n'exécutez pas `docker compose down --volumes`. Les versions sources restent disponibles via `compose.build.yaml`.

Dependabot est configuré dans le référentiel source pour proposer des mises à jour de dépendances ; il ne met pas automatiquement à niveau votre déploiement auto-hébergé. Testez à nouveau la connectivité du registre, la connexion et les actions concernées après une mise à niveau.
