# Déploiement automatique par SFTP (rsync over SSH)

Ce dépôt est un site statique. Le workflow [`sftp-deploy.yml`](./sftp-deploy.yml)
déploie automatiquement les fichiers servis (HTML, favicons, `logo.png`,
`robots.txt`, `sitemap.xml`, …) vers un serveur distant via **rsync over SSH**,
en s'authentifiant par **clé SSH** (jamais par mot de passe).

Il est **indépendant** des workflows Azure Static Web Apps présents dans
`.github/workflows/` : aucun de ces fichiers n'est modifié.

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

## 2. Générer une paire de clés SSH dédiée au déploiement

Créez une clé **dédiée** au déploiement (ne réutilisez jamais une clé
personnelle). L'action utilise rsync, qui n'accepte pas de passphrase :
**laissez la passphrase vide**.

```bash
ssh-keygen -m PEM -t ed25519 -C "github-actions-estampify-deploy" -f ./estampify_deploy
# (ou, si le serveur n'accepte pas ed25519 :)
# ssh-keygen -m PEM -t rsa -b 4096 -C "github-actions-estampify-deploy" -f ./estampify_deploy
```

Cela produit :

- `estampify_deploy` → **clé privée** (à mettre dans le secret `SFTP_SSH_PRIVATE_KEY`).
- `estampify_deploy.pub` → **clé publique** (à installer sur le serveur).

## 3. Installer la clé publique sur le serveur

Ajoutez le contenu de `estampify_deploy.pub` au fichier `~/.ssh/authorized_keys`
de l'utilisateur SFTP **sur le serveur** :

```bash
# Sur le serveur, connecté avec le compte de déploiement :
mkdir -p ~/.ssh && chmod 700 ~/.ssh
cat estampify_deploy.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

## 4. Récupérer la clé hôte (`known_hosts`) avec `ssh-keyscan`

Pour vérifier l'identité du serveur, récupérez son empreinte :

```bash
# Remplacez le port et l'hôte par les vôtres
ssh-keyscan -p 22 votre-serveur.example.com
```

Vérifiez l'empreinte affichée auprès de votre hébergeur, puis conservez cette
valeur (voir la section **Vérification de la clé hôte** ci-dessous pour son
usage et ses limites).

## 5. Créer les Secrets de l'Environment

Dans **Settings → Environments → production → Environment secrets**, ajoutez :

| Secret                  | Obligatoire | Description                                                        | Exemple                     |
| ----------------------- | ----------- | ------------------------------------------------------------------ | --------------------------- |
| `SFTP_HOST`             | Oui         | Nom d'hôte ou IP du serveur                                         | `votre-serveur.example.com` |
| `SFTP_USERNAME`         | Oui         | Utilisateur SSH/SFTP de déploiement                                 | `deploy`                    |
| `SFTP_SSH_PRIVATE_KEY`  | Oui         | Contenu **complet** de la clé privée `estampify_deploy`             | `-----BEGIN ... KEY-----`   |
| `SFTP_TARGET_DIR`       | Oui         | Dossier web distant cible                                          | `/var/www/html` ou `www/`   |
| `SFTP_PORT`             | Non (déf. 22) | Port SSH si différent de 22                                       | `2222`                      |

> ⚠️ Ne mettez **jamais** ces valeurs en clair dans le YAML. Elles ne sont
> référencées que via `${{ secrets.* }}`.

---

## Vérification de la clé hôte (host key) — usage et limite connue

Le workflow positionne `SSH_CMD_ARGS: "-o StrictHostKeyChecking=yes"` et déclenche
le remplissage de `known_hosts` via `ssh-keyscan` (grâce à `SCRIPT_BEFORE`).

**Limite importante de l'action `easingthemes/ssh-deploy` :** à chaque exécution,
l'action réinitialise le fichier `known_hosts`, puis le re-remplit automatiquement
via `ssh-keyscan`. Il s'agit donc d'un modèle **TOFU** (*Trust On First Use*)
ré-appliqué à chaque run : l'action ne permet pas d'**épingler** (pinning) une
empreinte vérifiée à l'avance de façon fiable. En pratique, la clé hôte présentée
par le serveur au moment du run est acceptée.

**Risque :** ce modèle ne protège pas contre un attaquant capable d'intercepter
la connexion (MITM) au moment précis du déploiement.

**Mitigations recommandées :**

- Utiliser un compte SFTP **dédié, chrooté** au seul dossier web (voir ci-dessous),
  ce qui limite l'impact d'une compromission.
- Restreindre le déploiement à la branche `main` et exiger une approbation
  (*required reviewers*) sur l'Environment `production`.
- Si une vérification stricte par empreinte épinglée est indispensable, remplacer
  l'action par un step `rsync`/`ssh` manuel : écrire la valeur de `ssh-keyscan`
  (étape 4, stockée dans un secret `SFTP_KNOWN_HOSTS`) dans `~/.ssh/known_hosts`
  **juste avant** d'appeler `rsync` avec `-e "ssh -o StrictHostKeyChecking=yes
  -o UserKnownHostsFile=~/.ssh/known_hosts"`, sans laisser un outil réécrire le
  fichier entre-temps.

---

## Recommandations de sécurité

- **Compte SFTP dédié et chrooté** : créez un utilisateur n'ayant accès qu'au
  dossier web (chroot/`ChrootDirectory`), sans shell interactif si possible.
- **Clé dédiée et sans passphrase** réservée à ce déploiement ; révoquez-la en
  supprimant la ligne correspondante de `authorized_keys` en cas de fuite.
- **Approbation manuelle** : activez *required reviewers* sur l'Environment.
- **Restriction de branche** : limitez les déploiements à `main`.
- **Permissions minimales** : le workflow déclare `permissions: contents: read`.
- **Action épinglée par SHA** : l'action est figée sur un commit précis
  (`5d8aeab1a56041e269567e67a08ce00df324a617`, v6.0.1) pour la sécurité
  supply-chain. Mettez à jour le SHA **et** le commentaire de version ensemble.

## Fichiers exclus du transfert

Les éléments suivants ne sont pas servis par le site et sont exclus via l'option
`EXCLUDE` de l'action :

`.git`, `.gitignore`, `.github`, `specs`, `.specify`, `README.md`, `LICENSE`,
`THIRD_PARTY_NOTICES.md`, `.ssh-deploy-state`.
