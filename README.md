# Matlab-Docker-Images

Images Docker basées exclusivement sur **MATLAB R2024a**, pour Linux `amd64`.

## Variantes et tags

Un seul dépôt Docker Hub (par exemple `moncompte/matlab`) peut contenir toutes
les variantes : le **tag**, après `:`, indique laquelle télécharger.

| Dossier de build | Image MathWorks parente | Tag proposé |
| --- | --- | --- |
| `base/` | `mathworks/matlab:r2024a` | `r2024a-base-v1` |
| `deep-learning/` | `mathworks/matlab-deep-learning:r2024a` | `r2024a-dl-v1` |
| `deep-learning-comms-signalblocks/` | `mathworks/matlab-deep-learning:r2024a` | `r2024a-dl-comms-signalblocks-v1` |

La variante enrichie conserve les produits de l’image deep learning et ajoute :

- `Communications_Toolbox` (Communications Toolbox, pas `communication_toolbox`) ;
- `DSP_System_Toolbox` (DSP System Toolbox, associé à la fonctionnalité de
  licence `Signal_Blocks`, pas à un produit MPM nommé `signal_blocks`) ;
- `Signal_Processing_Toolbox`, une dépendance requise ;
- `Simulink`, pour utiliser les bibliothèques de blocs.

L’installation utilise le [MATLAB Package Manager (MPM)](https://github.com/mathworks-ref-arch/matlab-dockerfile/tree/main/alternates/building-on-matlab-docker-image)
dans l’installation MATLAB existante. Les Dockerfiles conservent l’utilisateur
`matlab`, l’entrypoint et la commande par défaut des images parentes.
Le contenu deep learning est celui de [l’image officielle](https://github.com/mathworks-ref-arch/container-images/blob/main/matlab/README_DEEP_LEARNING.md) ;
aucune liste de produits supplémentaire n’était jointe à la demande.

La convention est `r2024a-<variante>-v<révision>`. Incrémenter la révision
(`v2`, `v3`, etc.) pour chaque nouvelle publication ; ne pas réutiliser un tag
déjà publié si l’on veut conserver les anciennes versions. Docker Hub permet
techniquement d’écraser un tag. Aucun tag `latest` n’est publié, afin de rendre
la variante et la version explicites. Les tags parents MathWorks restent
modifiables : une reconstruction peut donc récupérer une image parente mise à jour.

## Construire en local

Depuis la racine du dépôt, remplacer `moncompte/matlab` par votre dépôt Docker Hub :

```sh
docker build --platform linux/amd64 -t moncompte/matlab:r2024a-base-v1 ./base
docker build --platform linux/amd64 -t moncompte/matlab:r2024a-dl-v1 ./deep-learning
docker build --platform linux/amd64 -t moncompte/matlab:r2024a-dl-comms-signalblocks-v1 ./deep-learning-comms-signalblocks
```

Ces images et les téléchargements MPM sont volumineux : prévoir une connexion
Internet, suffisamment d’espace disque et du temps pour les builds.

## Publier sur Docker Hub avec GitHub Actions

1. Créer un dépôt Docker Hub, par exemple `moncompte/matlab`, avec la visibilité
   appropriée à vos droits de redistribution MathWorks.
2. Créer un jeton d’accès Docker Hub autorisé à écrire dans ce dépôt.
3. Dans GitHub, ouvrir **Settings → Secrets and variables → Actions** :
   - secrets : `DOCKERHUB_USERNAME` (identifiant Docker Hub) et
     `DOCKERHUB_TOKEN` (jeton d’accès) ;
   - variable : `DOCKERHUB_REPOSITORY`, par exemple `moncompte/matlab`
     (sans tag et sans `https://`).
4. Une fois le workflow présent sur la branche par défaut, ouvrir **Actions →
   Publish R2024a images to Docker Hub → Run workflow**. Choisir la branche
   contenant les Dockerfiles et saisir la révision numérique, par exemple `1`.

Le workflow construit puis publie les trois tags du tableau, sans nécessiter
de tag Git. Il est déclenché manuellement uniquement ; les pull requests ne
publient rien et n’utilisent pas les secrets Docker Hub. Les builds sont
séquentiels pour limiter la charge. Si le runner manque de disque pour les
images MATLAB, utiliser un runner disposant de plus de stockage en adaptant
`runs-on` dans le workflow.

Pour publier manuellement une image déjà construite :

```sh
docker login
docker push moncompte/matlab:r2024a-dl-comms-signalblocks-v1
```

Chaque tag se télécharge indépendamment :

```sh
docker pull moncompte/matlab:r2024a-dl-comms-signalblocks-v1
```

## Utiliser et vérifier une image

L’installation des produits **ne fournit pas de licence MATLAB**. Une licence
valide couvrant les produits utilisés reste nécessaire à l’exécution.
Vérifier également les conditions MathWorks avant toute redistribution,
notamment sur un dépôt Docker Hub public. La licence de ce dépôt ne remplace
pas les licences des logiciels MathWorks.

Exemple avec un serveur de licences réseau autorisé (remplacer son adresse) :

```sh
docker run --rm --platform linux/amd64 \
  -e MLM_LICENSE_FILE=27000@serveur-licences \
  moncompte/matlab:r2024a-dl-comms-signalblocks-v1 \
  -batch "assert(strcmp(version('-release'),'2024a')); ver; assert(~isempty(ver('comm'))); assert(~isempty(ver('dsp'))); assert(~isempty(ver('signal'))); assert(~isempty(ver('simulink')))"
```

Cette commande vérifie la release et la présence des produits ajoutés ; elle
ne garantit pas la disponibilité de chaque licence. Ne jamais inclure de
fichier de licence, de jeton ou d’identifiants dans les Dockerfiles, les
arguments de build ou les images.
