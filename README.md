# Bot QCM — Médecine (Telegram)

Bot Telegram de QCM (questions à choix multiples) organisés en arbre de
profondeur libre, avec :

- navigation par clavier persistant (module → dossier → … → questions) ;
- réponses correctes cachées sous un **spoiler** ;
- bouton **🚩 Signaler** sous chaque question : chaque signalement est envoyé
  en message privé aux admins et peut être **corrigé directement depuis
  Telegram** (texte, options, bonne réponse, suppression), avec commit GitHub
  optionnel pour survivre aux redéploiements ;
- génération d'un **examen HTML autonome** (correction en JavaScript) ;
- panneau admin (`/admin`, `/send`, `/reports`) ;
- abonnement obligatoire à une chaîne Telegram.

## Contenu

| Fichier | Rôle |
|---|---|
| `bot.py` | tout le bot (handlers, claviers, examen HTML) |
| `editor.py` / `editor.bat` | **éditeur local des questions** (navigateur, 127.0.0.1) |
| `qcm_storage.py` | chargement/sauvegarde des QCM depuis `data/*.json` |
| `render_app.py` | **point d'entrée Render** (serveur webhook aiohttp) |
| `wsgi_pythonanywhere.py` | point d'entrée PythonAnywhere (WSGI Flask) |
| `set_webhook.py` | configuration manuelle du webhook (PythonAnywhere) |
| `render.yaml` | blueprint Render (service + variables) |
| `requirements.txt` | dépendances Python |
| `runtime.txt` | version de Python pour Render |
| `DEPLOY_RENDER.md` | **guide de déploiement Render (recommandé)** |
| `DEPLOY_PYTHONANYWHERE.md` | ancien guide PythonAnywhere |

## Lancer en local

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:BOT_TOKEN="123456:ABC..."      # token @BotFather
python bot.py                        # mode polling (pas de webhook)
```

## Éditeur local des questions

Double-clique sur **`editor.bat`** (ou lance `python editor.py`) : le navigateur
s'ouvre sur <http://127.0.0.1:8000>. Tu peux alors :

- parcourir les modules → leçons → questions sans toucher au JSON ;
- ajouter / modifier / supprimer une question (validation : 2 à 5 options, au
  moins une bonne réponse) ;
- chercher dans toutes les questions ;
- cliquer sur **🚀 رفع إلى GitHub** : l'éditeur valide, commit et pousse les
  `data/*.json` modifiés — Render redéploie automatiquement (1–3 min).

Au démarrage, l'éditeur récupère la dernière version depuis GitHub
(`git pull --rebase`). Avant chaque sauvegarde, une copie est créée dans
`backups/` (ignoré par Git, 30 dernières copies par fichier).

Options utiles : `--port 8001`, `--no-sync` (ne pas contacter GitHub),
`--no-browser`, `--data-dir <dossier>`.

> Le bouton « 🚀 رفع » utilise **git** : la première fois, Git Credential
> Manager peut demander tes identifiants GitHub. Sans accès en écriture au
> dépôt, le fichier reste modifié localement (message 403 dans l'éditeur).

## Variables d'environnement

| Variable | Obligatoire | Défaut |
|---|---|---|
| `BOT_TOKEN` | oui | — |
| `ADMIN_USER_IDS` | non | `5696844039,5435592397` (séparés par des virgules) |
| `REQUIRED_CHANNEL` | non | `@qcMAX_batna` (vide = désactivé) |
| `CHANNEL_URL` | non | vide |
| `WEBHOOK_SECRET` | non | dérivé du token |
| `GITHUB_TOKEN` | non | vide (corrections Telegram non persistées) |
| `GITHUB_REPO` | non | `tomahzl55-byte/qcm5` |
| `GITHUB_BRANCH` | non | `main` |
| `PORT` / `RENDER_EXTERNAL_URL` | fournis par Render | — |

## Déploiement

Voir **`DEPLOY_RENDER.md`** (Render, gratuit, mode webhook).
