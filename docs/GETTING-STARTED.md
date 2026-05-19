# ATLAS NEXUS — Guide de Démarrage

> *Installez, configurez et exécutez votre première session ATLAS en moins de 15 minutes.*

---

## Prérequis

- **Python 3.10+** installé sur votre machine
- **Accès à au moins une API LLM** (OpenRouter recommandé — [openrouter.ai/keys](https://openrouter.ai/keys))
- **Git** pour cloner le dépôt
- Un terminal et une connexion internet

---

## Étape 1 : Installation

```bash
# Cloner le dépôt
git clone https://github.com/AtlasNexusTech/atlas-nexus.git
cd atlas-nexus

# Installer les dépendances
pip install -r requirements.txt

# Vérifier l'installation
python -c "import atlas; print('ATLAS prêt.')"
```

---

## Étape 2 : Configuration de Hermes

Hermes est l'agent coordinateur — c'est lui qui vous parle et orchestre les autres modules.

### 2.1 Fichier de configuration

Créez votre fichier d'environnement :

```bash
cp .env.example .env
```

Éditez `.env` et renseignez votre clé API :

```env
OPENROUTER_API_KEY=votre-clé-api-ici
```

### 2.2 Vérification

```bash
# Testez la connexion à l'API
python -m atlas.hermes --health-check
```

Résultat attendu :
```
✓ Connexion API OK
✓ Modèle chargé : anthropic/claude-opus-4.6
✓ Hermes prêt.
```

---

## Étape 3 : Votre première session ATLAS

### 3.1 Lancement

```bash
# Session interactive standard
python -m atlas.hermes --interactive
```

Vous verrez :

```
┌─────────────────────────────────────────┐
│  ATLAS NEXUS — Session interactive       │
│  Hermes à l'écoute. Tapez /aide.         │
└─────────────────────────────────────────┘
╰─>
```

### 3.2 Première requête : test simple

Posez une question simple pour vérifier le routage :

```
╰─> Quel est le meilleur langage pour un script d'automatisation, Python ou Bash ?
```

Hermes va :
1. **Évaluer** la complexité de la question
2. **Router** vers DELPHI pour l'analyse comparative
3. **Présenter** la réponse structurée avec options

### 3.3 Première décision structurée

Pour une décision qui mérite le protocole complet :

```
╰─> /decide Dois-je héberger ce service sur un VPS à 20€/mois ou utiliser un serverless ?
```

Hermes va :
1. **Cadrer** la décision (HERMES + DELPHI)
2. **Générer des options** avec analyse coût/bénéfice
3. **Stress-tester** les options via AEGIS
4. **Valider** la cohérence via ATHENA
5. **Présenter** la recommandation finale

---

## Étape 4 : Suivre le protocole de décision

Le protocole de décision ATLAS (`protocols/decision-protocol.md`) définit trois phases :

### Phase 1 — Cadrage (HERMES + DELPHI)
- Définir la décision à prendre
- Générer au minimum 3 options distinctes
- Construire un tableau comparatif pondéré

### Phase 2 — Stress-test (AEGIS)
- Audit des hypothèses : que se passe-t-il si chaque hypothèse est fausse ?
- Pré-mortem : « Supposons que la décision a échoué. Pourquoi ? »
- Score de risque : Impact × Probabilité × Réversibilité

### Phase 3 — Validation (ATHENA)
- Conformité aux règles établies
- Cohérence architecturale
- Vérification des standards de qualité

### Scores de risque et actions

| Score | Action |
|-------|--------|
| < 3   | Exécution autonome |
| 3–6   | Exécution avec vigilance, signalement à Hermes |
| 6–12  | Approbation humaine obligatoire |
| > 12  | Quorum complet requis (DELPHI + AEGIS + ATHENA) |

---

## Étape 5 : Journaliser les résultats

Chaque session ATLAS produit des artefacts qui sont stockés dans MNEMOSYNE :

```bash
# Consulter l'historique des décisions
python -m atlas.mnemosyne --list-decisions

# Consulter une décision spécifique
python -m atlas.mnemosyne --decision-id 42

# Exporter les décisions au format JSON
python -m atlas.mnemosyne --export decisions.json
```

### Ce qui est journalisé automatiquement

- **Décisions** : quoi, pourquoi, par qui, sous quelles hypothèses
- **Options évaluées** : chaque option avec scores et analyses
- **Désaccords** : quand AEGIS contredit DELPHI, le conflit est enregistré
- **Résultats** : succès, échec, corrections apportées

---

## Commandes essentielles

| Commande | Action |
|----------|--------|
| `/aide` | Affiche l'aide contextuelle |
| `/decide [question]` | Lance le protocole de décision complet |
| `/quick [question]` | Réponse rapide sans protocole lourd |
| `/status` | Affiche l'état du système et les modules actifs |
| `/history` | Affiche l'historique de la session |
| `/modules` | Liste les modules disponibles et leur état |
| `/quit` | Termine la session proprement |

---

## Résolution des problèmes courants

### « Connexion API refusée »

- Vérifiez votre clé API dans `.env`
- Vérifiez votre connexion internet
- Vérifiez les quotas sur votre compte OpenRouter

### « Module non disponible »

- Certains modules peuvent être désactivés en mode léger
- Utilisez `/modules` pour voir l'état de chaque module
- Forcez l'activation avec `--full` au lancement

### « Réponse trop lente »

- Le protocole complet mobilise plusieurs appels LLM
- Pour les questions simples, utilisez `/quick` au lieu de `/decide`
- Ajustez le modèle dans `config.yaml` (modèle plus léger = plus rapide)

---

## Prochaines étapes

1. **Lisez `ARCHITECTURE.md`** pour comprendre la structure complète du système
2. **Explorez `modules/`** pour comprendre le rôle de chaque agent
3. **Consultez `HERMES-INTEGRATION.md`** pour approfondir l'intégration Hermes-ATLAS
4. **Lisez `SELF-IMPROVEMENT-LOOP.md`** pour comprendre comment le système apprend
5. **Personnalisez** vos règles de gouvernance dans la configuration ATHENA

---

*ATLAS n'est pas un assistant. C'est un système. Prenez le temps de comprendre ses protocoles — c'est là que réside la valeur.*
