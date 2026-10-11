# Atelier DevOps — CI/CD en pratique

**Sujet :** construire sa première pipeline automatique sur une mini-calculatrice Python avec **GitHub Actions** (ou variante **GitLab CI**).

| | |
|---|---|
| **Durée** | ~2 h 30 (travail guidé) |
| **Niveau** | Débutant (terminal + Git de base) |
| **Outils** | Git, Python 3.9+, VS Code, compte GitHub ou GitLab |
| **Résultat attendu** | Une pipeline qui s’exécute à chaque `git push`, observable en graphique dans le navigateur |

---

## Objectifs pédagogiques

À l’issue du TP, vous devez être capable de :

- Expliquer **CI** (intégration continue) et **CD** (livraison / déploiement continus).
- Écrire un **fichier de pipeline** et comprendre chaque section.
- Lire une pipeline dans l’UI GitHub/GitLab : **jobs**, **étapes**, **logs**, **artefacts**.
- Constater qu’un **code cassé est bloqué** avant déploiement, puis le corriger.
- Ajouter une étape **Deploy** qui ne s’exécute que sur la **branche principale** (`main`).

---

## Pipeline cible

```
git push → Lint ──┐
                  ├──→ Build → Deploy
         Test ────┘
```

| Job | Rôle |
|-----|------|
| **Lint** | Vérification du style (flake8) |
| **Test** | Tests automatiques (pytest) — en **parallèle** avec Lint |
| **Build** | Construction d’un artefact (`dist/calculatrice.py`) — après succès de Lint **et** Test |
| **Deploy** | Mise en ligne **simulée** — uniquement si Build réussit **et** branche `main` |

Si un job échoue, les jobs suivants ne s’exécutent pas.

### Vocabulaire

| Terme | Définition |
|-------|------------|
| Pipeline | Enchaînement d’étapes déclenché par un événement |
| Job | Bloc de travail sur une machine propre |
| Step | Commande ou action dans un job |
| Trigger | Événement (push, PR, cron…) |
| Runner | Machine virtuelle GitHub/GitLab |
| Artefact | Fichier produit par un job et réutilisable |

---

## Déroulement et réalisations par partie

### Partie 0 — Préparer le poste (~10 min)

**À réaliser :**

1. Terminal opérationnel (PowerShell / Git Bash / Terminal).
2. `git --version` et `python --version` (Python ≥ 3.9).
3. Config Git : `user.name` et `user.email`.
4. Compte GitHub ou GitLab connecté dans le navigateur.
5. VS Code (ou équivalent).

**Contrôle :** Git et Python affichent une version ; compte distant accessible.

---

### Partie 1 — Comprendre le CI/CD (~15 min)

**À réaliser (sans code) :**

- Lire le schéma de la pipeline.
- Distinguer CI, livraison continue et déploiement continu.
- Identifier jobs parallèles vs séquentiels.
- Anticiper : si **Test** échoue, **Build** et **Deploy** ne partent pas.

**Contrôle :** savoir pourquoi **Deploy** est en dernier.

---

### Partie 2 — Projet calculatrice (~20 min)

**À réaliser :**

Créer le dossier `calculatrice-cicd` avec :

```
calculatrice-cicd/
├── calculatrice.py          # addition, soustraction, multiplication, division
├── test_calculatrice.py     # 5 tests pytest
├── requirements.txt         # pytest
├── .gitignore               # venv/, __pycache__/, .pytest_cache/
└── .github/workflows/
    └── pipeline.yml         # Partie 4
```

**Fonctions attendues dans `calculatrice.py` :**

- `addition`, `soustraction`, `multiplication`, `division` (erreur `ValueError` si division par zéro).

**Tests attendus :** 5 tests dont `test_division_par_zero` avec `pytest.raises(ValueError)`.

**Commandes locales :**

```bash
python -m pip install -r requirements.txt
python -m pytest -v
python -m pip install flake8
python -m flake8 . --max-line-length=100
```

**Contrôle :** `5 passed` ; flake8 sans message.

---

### Partie 3 — Dépôt distant (~15 min)

**À réaliser :**

1. Créer sur GitHub/GitLab un dépôt **vide** nommé `calculatrice-cicd` (sans README/.gitignore auto).
2. Initialiser Git et pousser le code :

```bash
git init
git add .
git commit -m "Premier commit : calculatrice et tests"
git branch -M main
git remote add origin https://github.com/VOTRE-NOM/calculatrice-cicd.git
git push -u origin main
```

**Note :** jeton GitHub avec permission **workflow** pour pousser des fichiers de pipeline.

**Contrôle :** les 4 fichiers visibles sur le dépôt distant.

---

### Partie 4 — Première pipeline GitHub Actions (~30 min)

**À réaliser :**

1. Créer `.github/workflows/pipeline.yml`.
2. Y copier la pipeline **Pipeline CI/CD Calculatrice** avec **4 jobs** : `lint`, `test`, `build`, `deploy`.
3. Déclencheurs : `push` et `pull_request` sur `main`.
4. Commit et push :

```bash
git add .github/workflows/pipeline.yml
git commit -m "Ajout de la pipeline CI/CD"
git push
```

**Points clés du YAML :**

- `needs: [lint, test]` sur **build**
- `needs: build` sur **deploy**
- `if: github.ref == 'refs/heads/main'` sur **deploy**
- Artefact `calculatrice-paquet` (upload dans build, download dans deploy)

**Contrôle :** onglet **Actions** — exécution « Pipeline CI/CD Calculatrice » (puis verte).

---

### Partie 5 — Observer la pipeline (~15 min)

**À réaliser :**

1. Ouvrir **Actions** → dernière exécution → graphique des 4 jobs.
2. Vérifier Lint + Test en parallèle, puis Build, puis Deploy.
3. Lire les logs du job **Tests automatiques** (`pytest -v`, 5 tests).
4. Télécharger l’artefact **calculatrice-paquet** (contient `calculatrice.py`).
5. Lire le log du job **Déploiement (simulé)**.

**Contrôle :** 4 jobs verts ; artefact téléchargé.

---

### Partie 6 — Casser puis réparer (~15 min)

**À réaliser :**

1. Bug volontaire : dans `addition`, remplacer `a + b` par `a - b`.
2. Vérifier en local : un test échoue.
3. Push malgré tout : commit « Bug volontaire dans l'addition ».
4. Constater job **Test** rouge ; **Build** et **Deploy** non exécutés (gris).
5. Corriger, `5 passed` en local, push « Correction de l'addition ».
6. Nouvelle exécution entièrement verte.

**Contrôle :** une exécution rouge puis une verte dans l’historique Actions.

---

### Partie 7 — Branche et Pull Request (~20 min)

**À réaliser :**

1. Branche `ajout-puissance` :

```bash
git switch -c ajout-puissance
```

2. Ajouter dans `calculatrice.py` :

```python
def puissance(a, b):
    return a ** b
```

3. Dans `test_calculatrice.py` : importer `puissance` et ajouter `test_puissance` (`assert puissance(2, 3) == 8`).
4. `python -m pytest -v` → **6 passed**.
5. Push de la branche et ouverture d’une **Pull Request** vers `main`.
6. Observer les checks : Lint, Test, Build OK ; **Deploy skipped** (pas sur `main`).
7. **Merge** la PR → pipeline sur `main` avec **Deploy** exécuté.
8. Local : `git switch main` puis `git pull`.

**Contrôle :** Deploy vert après merge ; `puissance` présente sur `main`.

---

### Partie 8 — Variante GitLab CI (~10 min, optionnel)

**À réaliser si GitLab :**

1. Projet vide `calculatrice-cicd` sur GitLab.
2. Fichier **`.gitlab-ci.yml` à la racine** (pas dans un sous-dossier).
3. Stages : `verification` (lint + test), `build`, `deploy`.
4. Règle deploy : `if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH`.
5. Push et consultation **Build > Pipelines** (graphique par stages).

**Contrôle :** pipeline GitLab verte avec 3 stages visibles.

| Notion | GitHub Actions | GitLab CI |
|--------|----------------|-----------|
| Fichier | `.github/workflows/*.yml` | `.gitlab-ci.yml` (racine) |
| Ordre | `needs` | `stages` |
| Deploy conditionnel | `if: github.ref == ...` | `rules: - if: ...` |
| Revue de code | Pull Request | Merge Request |

---

## Bilan — ce qui doit être en place

- [ ] Projet Python avec tests automatiques (puis 6 tests après la PR).
- [ ] Pipeline 4 jobs : **lint**, **test**, **build**, **deploy**.
- [ ] Déclenchement sur **push** et **pull_request** vers `main`.
- [ ] Blocage du déploiement si un test échoue.
- [ ] Visualisation graphique + logs + artefact.
- [ ] Workflow équipe : branche → PR → merge → deploy sur `main` uniquement.

---

## Questions de réflexion (à traiter en groupe ou rapport)

1. Différence entre intégration continue et déploiement continu ?
2. Pourquoi Deploy reste gris quand Test est rouge ?
3. Pourquoi Lint et Test en parallèle ?
4. Effet de supprimer `if: github.ref == 'refs/heads/main'` sur Deploy ?
5. Avantages pour une équipe de 10 personnes vs tests manuels ?

---

## Extensions facultatives

- Badge de statut CI dans le README.
- Couverture de tests (`pytest-cov`).
- Déploiement réel (GitHub Pages, Render, etc.).
- Image Docker dans le job Build.
- Notifications (e-mail, Discord, Slack) en cas d’échec.

---

## Aide-mémoire — problèmes fréquents

| Symptôme | Piste |
|----------|--------|
| Rien après push | Fichier absent de `.github/workflows/` ou mauvaise extension |
| Erreur YAML | Indentation (2 espaces, pas de tabulations) |
| Lint rouge (E302, E305…) | 2 lignes vides entre fonctions ; newline en fin de fichier |
| `ModuleNotFoundError` en Test | `requirements.txt` manquant ou non commité |
| Erreur jeton « workflow » | Recréer un PAT avec permission **workflow** |
| `remote origin already exists` | `git remote remove origin` puis reconfigurer |
| Actions désactivées | Activer les workflows (dépôts forkés) |
| GitLab stuck/pending | Runners partagés ou validation de compte |

---

## Captures d'écran pour `rapport-ci-cd.tex`

Enregistrez chaque capture **à la racine** du projet (`calculatrice-cicd/`), en **PNG**, avec le nom exact indiqué (sensible à la casse sous Linux : `PIPELINE.png` ≠ `pipeline.png`).

Fichiers **déjà présents** (vérifier qu'ils correspondent à la légende du rapport) :

| Fichier | Contenu attendu |
|---------|-----------------|
| `workflow.png` | Dossier `.github/workflows` dans VS Code (ou terminal) |
| `PIPELINE.png` | Contenu de `pipeline.yml` + commit/push du workflow |
| `github_actions.png` | Onglet **Actions** — liste des runs ou première exécution |

### À créer — Partie 2–3 (projet et dépôt)

| Fichier | Étape pour la capture |
|---------|------------------------|
| **`git.png`** | GitHub : page du dépôt avec les fichiers visibles (`calculatrice.py`, `test_calculatrice.py`, `requirements.txt`, `.gitignore`) après le premier push. *Alternative :* terminal montrant `git push` réussi + onglet du dépôt. |

### À créer — Partie 2 (local)

| Fichier | Étape pour la capture |
|---------|------------------------|
| **`pytest.png`** | Terminal : `python -m pytest -v` → **5 passed**. |
| **`flake8.png`** | Terminal : `python -m flake8 . --max-line-length=100` → **aucune ligne** de sortie (succès). Fenêtre entière ou zone terminal lisible. |

### À créer — Partie 5 (pipeline verte, 1er succès)

| Fichier | Étape pour la capture |
|---------|------------------------|
| **`pipeline-graph.png`** | GitHub **Actions** → cliquer sur une exécution **verte** → page **Summary** → graphique avec **Lint** et **Test** côte à côte, puis **Build**, puis **Deploy**. |
| **`pytest-actions-log.png`** | Même exécution → job **Tests automatiques** → déplier l'étape **Run pytest -v** → montrer les 5 tests passed. |
| **`artifact-calculatrice-paquet.png`** | Même exécution → bas de **Summary** → section **Artifacts** → artefact **`calculatrice-paquet`**. |
| **`deploy-log.png`** | Job **Déploiement (simulé)** → log avec `ls -l dist/` et le message *Application déployée avec succès !* |

### À créer — Partie 6 (bug et correction)

| Fichier | Étape pour la capture |
|---------|------------------------|
| **`pytest-bug-local.png`** | Après avoir mis `return a - b` dans `addition` : `pytest -v` → **1 failed** (`test_addition`). |
| **`pipeline-echec-test.png`** | Push du bug → **Actions** → run **rouge** → graphique ou détail : **Test** rouge, **Build** et **Deploy** gris / skipped. |
| **`pipeline-historique-rouge-vert.png`** | Liste des exécutions : une ligne **rouge** (commit bug) puis une **verte** (commit correction), visibles l'une sous l'autre. |

*Ordre des actions Partie 6 :*

1. Modifier `addition` → capture `pytest-bug-local.png`
2. `git commit` + `push` → attendre CI → capture `pipeline-echec-test.png`
3. Remettre `a + b`, push correction → capture `pipeline-historique-rouge-vert.png` (peut inclure les deux runs)

### À créer — Partie 7 (PR puissance)

| Fichier | Étape pour la capture |
|---------|------------------------|
| **`pytest-6-tests.png`** | Branche `ajout-puissance` : `pytest -v` → **6 passed**. |
| **`branche-ajout-puissance.png`** | GitHub : menu **Branches** ou page **Compare** montrant `ajout-puissance` et le commit *Ajout de la fonction puissance*. |
| **`pr-checks-deploy-skipped.png`** | Pull Request ouverte → section **Checks** / graphique CI : **Lint**, **Test**, **Build** verts et **Deploy** en **skipped** (gris). |
| **`pipeline-main-deploy-vert.png`** | Après **Merge** → nouvelle run sur **main** → graphique **4 jobs verts** incluant **Deploy**. |

### Compilation du PDF

```bash
cd calculatrice-cicd
pdflatex rapport-ci-cd.tex
pdflatex rapport-ci-cd.tex
```

Si une image manque, LaTeX affiche une erreur du type `File 'xxx.png' not found` : ajoutez le PNG ou commentez temporairement le bloc `\begin{figure}...\end{figure}` correspondant.

### Checklist rapide (17 fichiers)

- [ ] `git.png`
- [ ] `pytest.png`
- [ ] `flake8.png`
- [ ] `workflow.png` *(déjà là)*
- [ ] `PIPELINE.png` *(déjà là)*
- [ ] `github_actions.png` *(déjà là)*
- [ ] `pipeline-graph.png`
- [ ] `pytest-actions-log.png`
- [ ] `artifact-calculatrice-paquet.png`
- [ ] `deploy-log.png`
- [ ] `pytest-bug-local.png`
- [ ] `pipeline-echec-test.png`
- [ ] `pipeline-historique-rouge-vert.png`
- [ ] `pytest-6-tests.png`
- [ ] `branche-ajout-puissance.png`
- [ ] `pr-checks-deploy-skipped.png`
- [ ] `pipeline-main-deploy-vert.png`
