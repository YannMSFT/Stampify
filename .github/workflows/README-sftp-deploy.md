# Déploiement automatique par SFTP (auth par mot de passe)

Ce dépôt est un site statique. Le workflow [`sftp-deploy.yml`](./sftp-deploy.yml)
déploie automatiquement les fichiers servis (HTML, favicons, `logo.png`,
`robots.txt`, `sitemap.xml`, …) vers un serveur distant via **SFTP**, en
s'authentifiant par **mot de passe SSH** (secret `SFTP_PASSWORD`).

Il est **indépendant** des workflows Azure Static Web Apps présents dans
`.github/workflows/` : aucun de ces fichiers n'est modifié.

> ⚠️ **Sécurité — mot de passe vs clé SSH**
> L'authentification par mot de passe est **moins sûre** qu'une clé SSH : elle
> est sensible aux attaques par force brute et à l'interception. Si vous le
> pouvez, privilégiez une clé SSH. Si vous restez sur le mot de passe, utilisez
> impérativement un **mot de passe fort et unique** et un **compte SFTP dédié**
> (voir les recommandations en bas de page).

## Déclenchement

- À chaque `push` sur la branche `main`.
- Manuellement via **Actions → Deploy via SFTP → Run workflow** (`workflow_dispatch`).

Le bloc `concurrency` empêche deux déploiements simultanés ; un déploiement en
cours est annulé si un nouveau démarre.

---

## 1. Créer l'Environment GitHub `production`

Les secrets sont rattachés à un **Environment** protégeable :

1. **Settings → Environments → New environment** → nommez-le `production`.
2. (Recommandé) **Required reviewers** : ajoutez une ou plusieurs personnes qui
   devront approuver manuellement chaque déploiement.
3. (Recommandé) **Deployment branches** : limitez à `Selected branches` →
   `main`, pour qu'aucune autre branche ne puisse déployer.

## 2. Créer les Secrets de l'Environment

Dans **Settings → Environments → production → Environment secrets**, ajoutez :

| Secret              | Obligatoire   | Description                          | Exemple                     |
| ------------------- | ------------- | ------------------------------------ | --------------------------- |
| `SFTP_HOST`         | Oui           | Nom d'hôte ou IP du serveur          | `votre-serveur.example.com` |
| `SFTP_USERNAME`     | Oui           | Utilisateur SSH/SFTP de déploiement  | `deploy`                    |
| `SFTP_PASSWORD`     | Oui           | Mot de passe SSH du compte de déploiement | `••••••••••••`         |
| `SFTP_TARGET_DIR`   | Oui           | Dossier web distant cible            | `/var/www/html` ou `www/`   |
| `SFTP_PORT`         | Non (déf. 22) | Port SSH si différent de 22          | `2222`                      |

> ⚠️ Ne mettez **jamais** ces valeurs en clair dans le YAML. Elles ne sont
> référencées que via `${{ secrets.* }}`. Le mot de passe n'apparaît jamais dans
> les logs (les secrets sont masqués par GitHub Actions).

---

## Recommandations de sécurité

- **Mot de passe fort et unique** : long, aléatoire, jamais réutilisé ailleurs.
  Faites-le tourner régulièrement et révoquez-le immédiatement en cas de fuite.
- **Compte SFTP dédié et chrooté** : créez un utilisateur n'ayant accès qu'au
  dossier web (chroot/`ChrootDirectory`), sans shell interactif si possible.
  Cela limite fortement l'impact d'une compromission du mot de passe.
- **Approbation manuelle** : activez *required reviewers* sur l'Environment.
- **Restriction de branche** : limitez les déploiements à `main`.
- **Permissions minimales** : le workflow déclare `permissions: contents: read`.
- **Action épinglée par SHA** : l'action est figée sur un commit précis
  (`a5ccb9c6211a94cc59404f0fdb2a9936a6dfee64`, v1.2.6) pour la sécurité
  supply-chain. Mettez à jour le SHA **et** le commentaire de version ensemble.
- **Préférez une clé SSH** si votre serveur le permet : c'est l'option la plus
  sûre. Le mot de passe doit rester un choix par défaut acceptable, pas idéal.

## Fichiers exclus du transfert

Les éléments suivants ne sont pas servis par le site et sont exclus via l'option
`rsyncArgs` (`--exclude=…`) de l'action :

`.git`, `.gitignore`, `.github`, `specs`, `.specify`, `README.md`, `LICENSE`,
`THIRD_PARTY_NOTICES.md`.
