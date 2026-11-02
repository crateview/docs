---
title: "Opérations et suppression"
description: "Gérez soigneusement les manifestes et gardez le stockage du registre sous votre contrôle."
---

## Activer la suppression du manifeste

La suppression est désactivée par défaut dans l'interface utilisateur de Crateview. Le frontend et Distribution doivent le permettre :

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution peut également lire `REGISTRY_STORAGE_DELETE_ENABLED=true`. Redémarrez les services concernés après avoir modifié leur environnement ou leur configuration. Le registre reste responsable de l'autorisation, y compris des demandes effectuées directement via `/v2/**`.

## Comprendre la suppression de digest

L'opération de suppression standard cible un **manifeste digest**. Chaque balise pointant vers ce digest peut être supprimée. Crateview prévisualise les balises concernées avant la confirmation, mais un rédacteur simultané peut modifier les références après l'aperçu : le registre API ne fournit pas de transaction qui verrouille les alias.

Traitez une suppression comme une modification du manifeste et de tous ses alias, et non comme la suppression d'une étiquette. Vérifiez vos sauvegardes et la portée prévue avant de confirmer.


Sélectionnez un ou plusieurs tags du tableau et confirmez la suppression. L'aperçu recharge les tags affectés, déduplique les digests et signale les échecs partiels. Tous les alias sont concernés. Le nettoyage du stockage reste indépendant.

## Collecte des déchets et sauvegardes

La suppression d'un manifeste ne garantit pas une récupération immédiate du disque. La collecte des déchets est une tâche d'administration de registre distincte ; Crateview ne l'exécute pas.

Sauvegardez votre stockage de registre et sa configuration à l'aide d'une méthode appropriée au pilote de stockage. Conserver les volumes lors du remplacement des conteneurs. Suivez les instructions de Distribution pour le garbage collection et écrivez la coordination :

- [Distribution garbage collection](https://distribution.github.io/distribution/about/garbage-collection/)
- [Configuration Distribution](https://distribution.github.io/distribution/about/configuration/)

## Mettre à jour Crateview

Consultez les notes de version, définissez `CRATEVIEW_VERSION` sur la balise de version préfixée `v` souhaitée dans `.env` et remplacez le conteneur frontal :

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Par exemple, la version initiale utilise `CRATEVIEW_VERSION=v0.4.0`. Pour revenir en arrière, restaurez la balise précédente et répétez ces commandes. Conservez l'environnement d'exécution et le volume `registry-data` ; n'exécutez pas `docker compose down --volumes`. Les versions sources restent disponibles via `compose.build.yaml`.

Dependabot est configuré dans le référentiel source pour proposer des mises à jour de dépendances ; il ne met pas automatiquement à niveau votre déploiement auto-hébergé. Testez à nouveau la connectivité du registre, la connexion et les actions concernées après une mise à niveau.

## Grands registres

La recherche dans le référentiel lit le catalogue paginé complet. Les métadonnées des référentiels visibles sont récupérées à la demande. Le tri d'une grande liste de balises par heure de création d'image peut nécessiter la lecture de nombreux manifestes et configurations. Crateview n’a pas d’index de métadonnées d’arrière-plan ni de service de recherche externe.
