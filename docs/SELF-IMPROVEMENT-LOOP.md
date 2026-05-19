# ATLAS NEXUS — Boucle d'Auto-Amélioration

> *Comment le système apprend, se corrige et s'améliore au fil du temps : collecte de feedback, patterns de correction, consolidation mémoire, suivi des métriques et raffinement des protocoles.*

---

## Principe fondamental

ATLAS n'est pas figé. Chaque décision, chaque erreur, chaque correction alimente une boucle d'apprentissage qui rend le système plus performant avec le temps. Cette boucle est explicite, mesurable et vérifiable.

```
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│   ACTION ──▶ RESULTAT ──▶ FEEDBACK ──▶ CORRECTION ──▶      │
│      ▲                                                    │  │
│      │                                                    │  │
│      └────────── CONSOLIDATION ◀──── MÉTRIQUES ◀──────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 1. Collecte de feedback

Le feedback est la matière première de l'amélioration. ATLAS le collecte à trois niveaux.

### 1.1 Feedback humain explicite

L'utilisateur évalue les décisions et analyses :

```
╰─> /feedback decision-42
Évaluez cette décision (1-5) : 4
Commentaire : Bonne analyse mais a sous-estimé le coût de migration
```

```yaml
# Structure de feedback stockée dans MNEMOSYNE
feedback_entry:
  id: "fb-2026-05-20-001"
  decision_id: 42
  rating: 4
  comment: "Bonne analyse mais a sous-estimé le coût de migration"
  source: "human_explicit"
  timestamp: "2026-05-20T09:15:00Z"
```

### 1.2 Feedback implicite

Le système observe le comportement utilisateur sans demande explicite :

| Signal | Signification | Action |
|--------|---------------|--------|
| L'utilisateur ignore une recommandation | Désaccord silencieux | Baisser la confiance sur ce type d'analyse |
| L'utilisateur demande une reformulation | Sortie peu claire | Ajuster le niveau de détail |
| L'utilisateur fait l'inverse de la recommandation | Divergence forte | Enregistrer pour analyse |
| L'utilisateur exécute immédiatement | Confiance élevée | Renforcer le pattern |

### 1.3 Feedback automatique (métriques objectives)

Le système mesure ses propres performances :

```yaml
automatic_feedback:
  - metric: "decision_latency_seconds"
    target: "< 30"
    alert_if: "> 120"
  - metric: "option_diversity_score"
    target: "> 0.7"  # Les options sont-elles vraiment distinctes ?
    alert_if: "< 0.3"
  - metric: "assumption_coverage"
    target: "> 0.8"  # Proportion d'hypothèses auditées par AEGIS
    alert_if: "< 0.5"
  - metric: "human_override_rate"
    target: "< 0.2"  # L'humain ne devrait pas contredire souvent
    alert_if: "> 0.5"
```

---

## 2. Patterns de correction

Quand un problème est identifié, ATLAS applique des patterns de correction structurés.

### 2.1 Taxonomie des erreurs

| Catégorie | Exemple | Pattern de correction |
|-----------|---------|----------------------|
| **Erreur de cadrage** | Mauvaise compréhension du problème | Ajouter une étape de reformulation avant analyse |
| **Option manquante** | Solution évidente non proposée | Élargir le champ de recherche de DELPHI |
| **Angle mort AEGIS** | Risque non identifié | Ajouter une checklist spécifique au domaine |
| **Excès de prudence** | Score de risque systématiquement trop élevé | Recalibrer les seuils AEGIS |
| **Biais de confirmation** | Options qui se ressemblent trop | Forcer une option orthogonale systématique |
| **Hallucination factuelle** | Données incorrectes | Renforcer la vérification par source |

### 2.2 Processus de correction

```
DÉTECTION ──▶ CLASSIFICATION ──▶ ISOLATION ──▶ CORRECTION ──▶ VÉRIFICATION
    │               │                │              │               │
    │               │                │              │               │
    ▼               ▼                ▼              ▼               ▼
 Feedback    Catégorie        Identifier    Modifier le     Tester sur
  reçu       d'erreur         la racine     protocole       cas similaire
```

### 2.3 Exemple concret

**Problème** : DELPHI a proposé 3 options pour un investissement, mais elles variaient seulement sur le montant (100€, 200€, 300€) — même stratégie.

```
Détection : option_diversity_score = 0.2 (seuil : 0.7)
Classification : "Options non distinctes" → Option manquante
Isolation : DELPHI n'a pas exploré de classe d'actifs différente
Correction : Ajout d'une règle ATHENA : "Toute analyse d'investissement doit inclure
             au moins une option dans une classe d'actifs différente"
Vérification : Test sur 5 décisions d'investissement passées → diversité remontée à 0.8
```

---

## 3. Consolidation mémoire

MNEMOSYNE ne stocke pas seulement — elle consolide, indexe et rend découvrables les apprentissages.

### 3.1 Niveaux de mémoire

```
┌─────────────────────────────────────────────┐
│ MÉMOIRE DE TRAVAIL (session)                 │
│ ↕ Consolidation toutes les 10 interactions  │
├─────────────────────────────────────────────┤
│ MÉMOIRE À COURT TERME (7 jours)             │
│ ↕ Consolidation quotidienne                 │
├─────────────────────────────────────────────┤
│ MÉMOIRE À LONG TERME (permanente)           │
│ Patterns, heuristiques, corrections          │
└─────────────────────────────────────────────┘
```

### 3.2 Processus de consolidation

Chaque nuit (ou après chaque session), MNEMOSYNE exécute :

```python
# Pseudo-code du consolidateur MNEMOSYNE
def consolidate():
    recent_decisions = get_decisions_since(last_consolidation)
    
    for decision in recent_decisions:
        # 1. Identifier les patterns
        if decision.similar_to(previous_failures):
            flag_as_pattern(decision)
        
        # 2. Mettre à jour les heuristiques
        if decision.outcome == SUCCESS:
            reinforce_heuristic(decision.strategy)
        elif decision.outcome == FAILURE:
            weaken_heuristic(decision.strategy)
        
        # 3. Générer des résumés
        summary = generate_summary(decision)
        store_summary(summary)
        
        # 4. Lier les décisions connexes
        link_related_decisions(decision)
    
    # 5. Élaguer l'ancien
    prune_old_short_term_memories(older_than_days=7)
```

### 3.3 Ce qui est consolidé

| Type | Contenu | Fréquence | Durée de vie |
|------|---------|-----------|-------------|
| **Décisions** | Quoi, pourquoi, résultat | Immédiate | Permanent |
| **Patterns** | Séquences récurrentes (succès/échec) | Quotidienne | Permanent |
| **Heuristiques** | Règles empiriques validées | Hebdomadaire | Jusqu'à contradiction |
| **Corrections** | Modifications de protocole | Immédiate | Permanent |
| **Métriques** | Scores, tendances | Quotidienne | 90 jours |

---

## 4. Suivi des métriques

ATLAS mesure sa performance pour détecter les dérives et prouver son amélioration.

### 4.1 Tableau de bord des métriques clés

| Métrique | Cible | Alerte | Fréquence |
|----------|-------|--------|-----------|
| **Taux de décisions correctes** | > 85% | < 70% | Hebdomadaire |
| **Taux d'approbation humaine** | > 80% | < 60% | Mensuel |
| **Taux d'override humain** | < 20% | > 40% | Mensuel |
| **Diversité des options** | > 0.7 | < 0.3 | Par décision |
| **Latence moyenne** | < 30s | > 120s | Quotidien |
| **Taux d'erreurs AEGIS** (faux positifs) | < 10% | > 25% | Mensuel |
| **Couverture des hypothèses** | > 80% | < 50% | Par décision |
| **Satisfaction utilisateur** | > 4/5 | < 3/5 | Hebdomadaire |

### 4.2 Visualisation des tendances

```bash
# Afficher les tendances des 30 derniers jours
python -m atlas.mnemosyne --metrics --last 30d

# Résultat :
# ┌────────────────────────────────────────────────────┐
# │ MÉTRIQUES — 30 jours                               │
# ├──────────────────────────┬────────┬────────────────┤
# │ Métrique                 │ Valeur │ Tendance       │
# ├──────────────────────────┼────────┼────────────────┤
# │ Décisions correctes      │ 88%    │ ▲ +3%          │
# │ Approbation humaine      │ 82%    │ ▲ +5%          │
# │ Diversité des options    │ 0.76   │ ▶ stable       │
# │ Latence moyenne          │ 22s    │ ▼ -4s          │
# │ Satisfaction             │ 4.2/5  │ ▲ +0.3         │
# └──────────────────────────┴────────┴────────────────┘
```

### 4.3 Alertes automatiques

```yaml
# config/monitoring.yaml
alerts:
  - name: "degradation_decision"
    condition: "decision_accuracy_7d < 0.70"
    action: "notify_human + trigger_diagnostic"
    message: "⚠️ Le taux de décisions correctes est tombé à {value}. Diagnostic automatique lancé."
  
  - name: "derive_protocole"
    condition: "protocol_adherence_7d < 0.80"
    action: "notify_human"
    message: "⚠️ Le protocole de décision n'est pas suivi dans {gap}% des cas."
  
  - name: "boucle_stagnation"
    condition: "decision_accuracy_30d_volatility < 0.02 AND decision_accuracy_30d < 0.90"
    action: "suggest_protocol_review"
    message: "ℹ️ Performance stable mais sous l'objectif. Une revue de protocole est suggérée."
```

---

## 5. Raffinement des protocoles

Les protocoles ATLAS ne sont pas gravés dans le marbre. Ils évoluent par corrections successives documentées.

### 5.1 Cycle de vie d'un protocole

```
v1.0 ──▶ v1.1 ──▶ v1.2 ──▶ ... ──▶ v2.0
 │         │         │                │
 │         │         │                └── Refonte : le protocole
 │         │         │                    a changé de paradigme
 │         │         │
 │         │         └── Correction mineure
 │         │              (ex: seuil ajusté)
 │         │
 │         └── Amélioration ciblée
 │              (ex: étape ajoutée)
 │
 └── Version initiale
```

### 5.2 Règles d'évolution

1. **Toute modification est justifiée** — chaque changement de protocole référence le feedback ou la métrique qui l'a déclenché
2. **Une modification à la fois** — on ne change pas deux variables simultanément
3. **Période d'observation** — après un changement, on observe les métriques pendant 7 jours avant de conclure
4. **Droit de veto humain** — les changements de protocole qui modifient les seuils de risque (> 20%) nécessitent une approbation explicite
5. **Journal d'évolution** — chaque version est documentée dans `protocols/CHANGELOG.md`

### 5.3 Exemple d'évolution documentée

```markdown
## Protocole de décision — Journal d'évolution

### v1.3 (2026-05-15)
- **Ajout** : Étape 2.4 « Vérification croisée des sources » après 3 cas d'hallucination factuelle
- **Déclencheur** : fb-2026-05-10 (données incorrectes dans analyse investissement)
- **Métrique avant** : précision factuelle 91% → **après** : 97%

### v1.2 (2026-04-28)
- **Modification** : Seuil de risque « exécution autonome » relevé de 3 à 4
- **Déclencheur** : 12 cas où le score 3 nécessitait une approbation inutilement (override humain à 95%)
- **Métrique avant** : override inutile 15% → **après** : 5%

### v1.1 (2026-04-10)
- **Ajout** : Option D orthogonale obligatoire pour toute analyse stratégique
- **Déclencheur** : métrique diversité_options passée de 0.7 à 0.5 en mars
- **Métrique avant** : diversité 0.5 → **après** : 0.78

### v1.0 (2026-03-15)
- Version initiale du protocole de décision
```

---

## 6. Boucle complète : un exemple concret

Voici comment une erreur se transforme en amélioration permanente :

```
JOUR 1 — L'ERREUR
├── DELPHI propose "Allouer 100% du budget à l'ETF Monde"
├── AEGIS ne détecte pas le risque de concentration
├── ATHENA valide (l'ETF Monde est un produit conforme)
└── Décision exécutée

JOUR 7 — LE FEEDBACK
├── L'utilisateur lit une analyse sur les risques de concentration
├── Feedback explicite : "AEGIS aurait dû signaler le risque de concentration 
│   géographique (70% USA dans l'ETF Monde)"
└── Note : 3/5

JOUR 7 — LA CORRECTION
├── Analyse automatique : pourquoi AEGIS n'a pas détecté ?
├── Cause : AEGIS vérifiait les hypothèses explicites, pas le non-dit
│   (« diversifié » n'était pas remis en question)
└── Correction : Ajout d'une checklist « Risques standards » dans AEGIS :
    - Concentration géographique
    - Concentration sectorielle
    - Risque de change
    - Liquidité du sous-jacent

JOUR 14 — LA CONSOLIDATION
├── MNEMOSYNE consolide : pattern de décision d'allocation + correction AEGIS
├── Heuristique créée : "Toute allocation > 50% dans un seul véhicule → 
│   vérification obligatoire de concentration"
└── Métriques mises à jour

JOUR 30 — LA VÉRIFICATION
├── 3 nouvelles décisions d'allocation depuis la correction
├── AEGIS a correctement signalé le risque de concentration dans les 3 cas
├── Métrique « couverture des hypothèses » : 72% → 91%
└── Correction validée et intégrée dans le protocole v1.4
```

---

## Commandes de la boucle d'amélioration

| Commande | Action |
|----------|--------|
| `/feedback [id]` | Donner un feedback sur une décision |
| `/metrics` | Afficher le tableau de bord des métriques |
| `/trends` | Voir les tendances sur 30 jours |
| `/review-protocol` | Suggérer des améliorations de protocole |
| `/learnings` | Voir les apprentissages consolidés récents |
| `/health` | Diagnostic complet du système |

---

## Principes directeurs

1. **Pas d'apprentissage sans trace** — tout apprentissage est documenté et lié à sa source
2. **Pas de correction sans vérification** — on mesure l'effet de la correction avant de la valider
3. **Pas d'évolution silencieuse** — l'utilisateur est informé quand les protocoles changent
4. **La transparence avant la performance** — mieux vaut une correction visible qu'une optimisation opaque
5. **L'humain a le dernier mot** — la boucle automatique peut suggérer, jamais imposer

---

*L'auto-amélioration n'est pas une fonctionnalité — c'est la caractéristique qui distingue ATLAS d'un simple outil. Le système qui apprend est le système qui dure.*
