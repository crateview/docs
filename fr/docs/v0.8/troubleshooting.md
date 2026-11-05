---
title: "Dépannage"
description: "Résolvez les problèmes de connexion, d’authentification, de métadonnées et de déploiement."
---

## L'interface utilisateur ne peut pas accéder au registre

Vérifiez l'origine du registre à partir de l'environnement réseau du serveur frontal. Dans un conteneur, `localhost` pointe vers le conteneur lui-même. Un Compose courant en amont est `http://registry:5000`.

```sh
curl -i http://127.0.0.1:5000/v2/
```

Un registre anonyme devrait réussir. Un authentifié peut renvoyer `401` avec un défi `WWW-Authenticate`. Un refus de connexion signifie que l'écouteur, l'adresse ou le chemin réseau n'est pas prêt.

## La connexion échoue

Crateview prend en charge les registres anonymes, le service natif htpasswd Basic Auth et le service optionnel Crateview auth en mode jeton. Il sonde `/v2/` et affiche le formulaire de connexion lorsque le registre requiert une authentification de base.

Les services de jetons Bearer externes et OIDC ne sont pas intégrés. Les services de jetons externes non configurés génèrent une notification explicite non prise en charge-auth.

En cas de perte de session répétée, vérifiez que toutes les instances frontales partagent `NUXT_SESSION_PASSWORD`. Un secret généré par un processus local change après un redémarrage. Vérifiez également les en-têtes HTTPS et de protocole transféré ; les sessions du navigateur nécessitent que le cookie revienne à la même origine.

## Mon image n'a pas de description ni de date de création

Ces valeurs doivent exister dans les annotations du manifeste ou la configuration de l'image. Crateview ne déduit pas les métadonnées manquantes et ne remplace pas les heures push du registre. Ajoutez les annotations OCI appropriées lors de la création de vos images.

## Les référents sont vides

Le registre doit exposer le point de terminaison des référents OCI ou l’index de secours de la balise digest standard. Les clients OCI sont responsables de la maintenance de cet index de secours lorsque cela est nécessaire. Le simple fait de transférer un manifeste portant un sujet dans un registre sans prise en charge des référents ne permet pas à Crateview d'analyser toutes les balises à sa recherche.

## La suppression est manquante ou rejetée

Activer les deux Crateviewc'est `NUXT_REGISTRY_DELETE_ENABLED` et Distributionle paramètre de suppression de stockage de . Le registre peut toujours rejeter la demande en fonction de ses autorisations ou de sa mise en œuvre. La suppression fonctionne par digest et peut affecter plusieurs balises.

## La production ignore mon fichier .env

`node .output/server/index.mjs` ne charge pas automatiquement `.env`. Exportez des variables via votre gestionnaire de processus, votre environnement de conteneur ou votre shell. Redémarrez après avoir modifié les valeurs d'exécution. Un hébergeur Web uniquement statique ne peut pas remplacer le serveur Nitro requis.

## La commande pull copiée ne fonctionne pas

La commande utilise l'adresse publique du frontend. Confirmez que la CLI peut atteindre cette origine et fait confiance à son certificat TLS. Authentifiez la CLI séparément si nécessaire. Une connexion au navigateur ne crée pas de connexion CLI Docker.

Si vous trouvez un problème d'interface utilisateur reproductible, ouvrez un [issue](https://github.com/crateview/registry-ui/issues) avec la révision concernée et les détails de reproduction aseptisés. Utilisez le [canal de sécurité privé](https://github.com/crateview/registry-ui/security/policy) pour les vulnérabilités.
## Supervision

Configuration de cette version: [Supervision](/fr/docs/v0.8/monitoring).

Configuration de cette version — La supervision est facultative et désactivée par défaut. La vue d'ensemble affiche le tableau de bord si elle est active ; sinon /home redirige vers /explore. L'activer expose les données globales à toute personne ayant accès à l'interface, y compris les visiteurs anonymes des registres publics. Le mode à jetons géré exige aussi un droit de lecture global.

## L’installation PWA ou le démarrage hors ligne échoue

Vérifiez la prise en charge du navigateur, HTTPS ou localhost et le build de production. Visitez d’abord l’application en ligne et laissez le worker s’installer. Les requêtes pour `manifest.webmanifest` et `sw.js` doivent atteindre Crateview sous `NUXT_APP_BASE_URL` ; le proxy inverse doit conserver ce préfixe et fournir les bonnes réponses de manifeste et JavaScript.

Le message hors ligne est attendu sans réseau ; il ne fournit aucune donnée de dépôt ou de compte enregistrée. Reconnectez-vous et réessayez. Si les icônes sont incorrectes, vérifiez les dimensions PNG exactes, les chemins lisibles dans le conteneur, la priorité des fichiers et les erreurs au démarrage ; redémarrez après modification. Le navigateur peut actualiser les icônes plus tard.
