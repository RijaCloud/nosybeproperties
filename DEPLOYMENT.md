# Publication sur Nginx

Les liens internes utilisent des URL sans extension. Les fichiers HTML restent
 en place : `/about` doit servir `about.html`.

## Installation

1. Publier les fichiers du site a la racine du domaine.
2. Copier `nginx/clean-urls.conf` vers
   `/etc/nginx/snippets/nosybe-clean-urls.conf` sur le serveur.
3. Dans le bloc `server {}` qui sert le site, conserver les reglages de domaine,
   HTTPS et `root`. Remplacer le bloc `location /` existant par :

   ```nginx
   include /etc/nginx/snippets/nosybe-clean-urls.conf;
   ```

   Placer cet include directement dans `server {}`. Remplacer aussi un eventuel
   bloc `location = /` existant pour eviter un doublon. Verifier les autres
   blocs `location` ciblant les fichiers HTML ou certains chemins : ils peuvent
   prendre priorite. Le `root` doit designer le dossier contenant `index.html`.

4. Verifier et recharger :

   ```sh
   sudo nginx -t && sudo systemctl reload nginx
   ```

Nginx ne charge pas automatiquement une configuration deposee dans le dossier du
site. Un simple push ne suffit donc pas a activer ces regles. La configuration
Apache `.htaccess` a ete retiree.

## Verification apres activation

- `/` affiche l'accueil ; `/index.html` redirige vers `/`.
- `/about.html?source=test` redirige en HTTP 301 vers `/about?source=test`.
- `/about` fonctionne en acces direct et apres rafraichissement.
- `/property/modern-family-home` affiche la fiche correspondante.
- `/agent`, `/blog` et `/property` affichent leurs pages malgre les dossiers
  portant les memes noms.
- Une page inexistante renvoie HTTP 404.
- Les images, styles et scripts restent accessibles.

Les parametres de requete sont conserves. Ces regles sont prevues pour un site
statique servi a la racine du domaine.

Reference : https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files

## Ressources du site

Le dossier `_external/` est inclus dans le depot. Le publier au meme niveau
que `index.html` : les pages utilisent des chemins `/_external/...`.
La configuration Nginx fournie sert ces fichiers sans regle supplementaire.
Le composant JavaScript local inclut la suppression des boutons promotionnels.
Ses dependances Framer restent chargees depuis leur CDN d'origine ; cette
copie du site ne constitue donc pas une version entierement hors ligne.

Le script de statistiques Framer a ete retire des pages : sa copie locale
calculait son point de collecte a partir du domaine du site et provoquait
un POST /anonymous en erreur. Les imports JavaScript statiques et dynamiques
du composant local pointent vers les modules disponibles sur le CDN Framer.
