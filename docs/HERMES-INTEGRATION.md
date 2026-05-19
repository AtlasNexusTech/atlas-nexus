# HERMES — Intégration ATLAS

> *Comment l'agent Hermes implémente l'architecture ATLAS : cartographie des compétences, configuration des invites, injection de contexte et déclencheurs d'automatisation.*

---

## Vue d'ensemble

Hermes n'est pas un simple chatbot. C'est la **couche de coordination** qui transforme une intention humaine en un processus ATLAS structuré. Il interprète, évalue, route, arbitre et consolide.

```
┌──────────────────────────────────────────────────┐
│                 UTILISATEUR                       │
│                      │                            │
│                      ▼                            │
│  ┌──────────────────────────────────────┐        │
│  │           HERMES                      │        │
│  │  ┌─────────┐  ┌─────────┐  ┌───────┐ │        │
│  │  │ Intake  │  │ Routage │  │Arbitre│ │        │
│  │  └─────────┘  └─────────┘  └───────┘ │        │
│  └──────┬──────────┬──────────┬─────────┘        │
│         │          │          │                   │
│         ▼          ▼          ▼                   │
│    DELPHI      AEGIS      ATHENA                  │
│         │          │          │                   │
│         └──────────┴──────────┘                   │
│                    │                              │
│                    ▼                              │
│              MNEMOSYNE                            │
└──────────────────────────────────────────────────┘
```

---

## Cartographie des compétences

Chaque module ATLAS correspond à une compétence (skill) activable par Hermes. La correspondance est directe :

### Table de correspondance

| Module ATLAS | Compétence Hermes | Déclencheur | Fonction |
|-------------|-------------------|-------------|----------|
| **DELPHI** | `deep_synthesis` | Requête analytique, demande comparative | Synthèse, modélisation, génération d'options |
| **AEGIS** | `critical_validation` | Décision proposée, assertion à vérifier | Stress-test, contre-arguments, audit |
| **ATHENA** | `governance_check` | Changement structurel, validation finale | Règles, standards, cohérence |
| **HEPHAESTUS** | `infrastructure` | Déploiement, tâche automatisée | Cycle de vie, monitoring, pipelines |
| **MNEMOSYNE** | `memory_ops` | Chaque action significative | Stockage, indexation, récupération |

### Chaînage des compétences

Hermes peut enchaîner plusieurs compétences séquentiellement ou en parallèle :

```
Requête : "Devrais-je migrer ma base de données vers PostgreSQL ?"

Hermes évalue :
├── Complexité : ÉLEVÉE (décision technique avec trade-offs)
├── Risque : MOYEN (migration réversible mais coûteuse)
└── Routage : DELPHI → AEGIS → ATHENA

Exécution :
1. DELPHI — deep_synthesis
   → Analyse PostgreSQL vs SQLite : performances, coûts, compatibilité
   → Génération de 3 options : migration complète, hybride, statu quo

2. AEGIS — critical_validation
   → Audit des hypothèses de DELPHI
   → Pré-mortem : "La migration a échoué — pourquoi ?"
   → Score de risque calculé

3. ATHENA — govern_ance_check
   → La migration respecte-t-elle les standards de l'infrastructure ?
   → Impact sur la cohérence architecturale

4. HERMES — consolidation
   → Présentation des options avec scores, risques, et recommandation
```

---

## Configuration des invites (prompts)

Hermes utilise un système d'invites structurées à plusieurs niveaux.

### 1. Invite système (System Prompt)

L'invite système définit l'identité et les règles de fonctionnement de Hermes :

```yaml
# config/hermes.yaml
system_prompt:
  identity: |
    Tu es Hermes Agent, le coordinateur du système ATLAS NEXUS.
    Tu ne réponds pas — tu orchestres, analyses et structures.
  core_rules:
    - "Toujours évaluer la complexité AVANT de répondre"
    - "Pour une décision non triviale, suivre le protocole de décision ATLAS"
    - "Ne jamais donner d'avis personnel déguisé en analyse"
    - "Présenter les désaccords entre modules, pas les cacher"
  language:
    default: "Français (langue de l'utilisateur)"
    technical_terms: "Anglais autorisé pour les termes techniques"
```

### 2. Invites de routage (Routing Prompts)

Pour chaque compétence, un pré-prompt spécialisé est injecté :

```yaml
routing_prompts:
  deep_synthesis: |
    [MODE DELPHI ACTIVÉ]
    Tu analyses en profondeur. Tu produis des OPTIONS, pas des paragraphes.
    Format obligatoire : tableau comparatif avec critères pondérés.
    Minimum 3 options distinctes. Quantifie quand c'est possible.

  critical_validation: |
    [MODE AEGIS ACTIVÉ]
    Tu es l'avocat du diable. Ton rôle : trouver ce qui peut échouer.
    Pour chaque hypothèse : quel est le pire scénario si elle est fausse ?
    Format : Tableau des risques avec scores Impact × Probabilité × Réversibilité.

  govern_ance_check: |
    [MODE ATHENA ACTIVÉ]
    Tu vérifies la conformité. Règles > préférences.
    Points de contrôle : standards, cohérence, sécurité, maintenabilité.
    Si non-conforme : tu bloques et expliques pourquoi.
```

### 3. Invites d'arbitrage (Arbitration Prompts)

Quand deux modules sont en désaccord :

```yaml
arbitration_prompt: |
  [ARBITRAGE HERMES]
  Conflit détecté entre {module_a} et {module_b}.
  Point de désaccord : {disagreement_point}
  
  Tâche :
  1. Reformuler clairement les deux positions
  2. Identifier la racine du désaccord (fait, valeur, hypothèse ?)
  3. Si résoluble automatiquement : trancher avec justification
  4. Si non résoluble : formuler la question exacte à poser à l'humain
```

---

## Injection de contexte

Hermes maintient la continuité cognitive en injectant du contexte structuré à chaque appel.

### Sources de contexte

```
┌─────────────────────────────────────────────┐
│           CONTEXTE INJECTÉ                   │
├─────────────────────────────────────────────┤
│ 1. Décisions précédentes (MNEMOSYNE)         │
│    └── 5 dernières décisions pertinentes     │
│ 2. Règles actives (ATHENA)                   │
│    └── Règles de gouvernance applicables     │
│ 3. État du système (HEPHAESTUS)              │
│    └── Modules actifs, ressources dispo      │
│ 4. Historique de session                     │
│    └── Derniers échanges (fenêtre glissante) │
│ 5. Préférences utilisateur                   │
│    └── Langue, niveau de détail, style       │
└─────────────────────────────────────────────┘
```

### Fenêtre de contexte glissante

```python
# Concept d'implémentation
class ContextWindow:
    def __init__(self, max_tokens=8000):
        self.max_tokens = max_tokens
        self.system_prompt = load_system_prompt()
        self.memory_entries = []   # Depuis MNEMOSYNE
        self.session_history = []  # Échanges récents
        self.active_rules = []     # Depuis ATHENA
    
    def build_context(self, current_task):
        """Assemble le contexte complet pour l'appel LLM."""
        context = self.system_prompt
        
        # Priorité 1 : règles actives (ATHENA)
        context += format_rules(self.active_rules)
        
        # Priorité 2 : mémoire pertinente (MNEMOSYNE)
        relevant_memories = retrieve_similar(current_task)
        context += format_memories(relevant_memories)
        
        # Priorité 3 : historique récent
        context += format_history(self.session_history[-10:])
        
        return context
```

---

## Déclencheurs d'automatisation

Hermes peut agir de manière proactive via des déclencheurs configurés.

### Types de déclencheurs

| Type | Exemple | Action |
|------|---------|--------|
| **Temporel** | Tous les lundis à 9h | Bilan hebdomadaire + suggestions |
| **Événementiel** | Alerte de monitoring | Analyse d'incident + propositions |
| **Seuil** | Portfolio -5% en 24h | Analyse de risque + options |
| **Manuel** | Commande `/review` | Revue complète de l'état du système |

### Configuration

```yaml
# config/triggers.yaml
triggers:
  - name: "revue_hebdomadaire"
    type: temporal
    schedule: "0 9 * * 1"  # Lundi 9h
    action:
      modules: [DELPHI, MNEMOSYNE]
      prompt: "Revue hebdomadaire : décisions de la semaine, anomalies, suggestions"
    
  - name: "alerte_performance"
    type: event
    source: "hephaestus.monitoring"
    condition: "error_rate > 5%"
    action:
      modules: [DELPHI, AEGIS]
      prompt: "Analyse de l'alerte performance. Cause racine ? Actions correctives ?"
    
  - name: "verification_portefeuille"
    type: threshold
    metric: "portfolio.daily_change_pct"
    condition: "< -5"
    action:
      modules: [DELPHI, AEGIS, ATHENA]
      prompt: "Analyse de la baisse du portefeuille. Panique ou opportunité ?"
      require_approval: true
```

### Boucle d'automatisation

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ TRIGGER  │ ──▶ │ HERMES   │ ──▶ │ MODULES  │
│ (event)  │     │ (évalue) │     │ (action) │
└──────────┘     └──────────┘     └──────────┘
                       │                 │
                       │  Haute confiance│
                       │  ┌──────────┐   │
                       └─▶│ Exécution │◀──┘
                          │ autonome  │
                          └──────────┘
                       │
                       │  Doute
                       │  ┌──────────┐
                       └─▶│ Demande   │
                          │ humaine   │
                          └──────────┘
```

---

## Bonnes pratiques

### Pour les développeurs intégrant Hermes

1. **Toujours passer par Hermes** — ne jamais appeler les modules directement. Hermes garantit la traçabilité et l'arbitrage.

2. **Respecter les scores de risque** — le score produit par AEGIS n'est pas indicatif. Si le score est > 6, l'approbation humaine est obligatoire.

3. **Journaliser chaque décision** — MNEMOSYNE doit recevoir l'entrée APRÈS chaque décision, pas avant.

4. **Ne pas contourner ATHENA** — même les décisions apparemment anodines peuvent violer une règle de gouvernance.

5. **Préférer la clarté à la vitesse** — un protocole bien exécuté prend quelques appels API de plus, mais évite des erreurs coûteuses.

### Pour les utilisateurs

1. **Soyez explicite** — « Que dois-je faire ? » → Hermes va demander des précisions. « Dois-je choisir A ou B pour [contexte précis] ? » → Hermes peut agir directement.

2. **Utilisez `/decide` pour les vraies décisions** — la commande force le protocole complet, même si Hermes pensait pouvoir répondre rapidement.

3. **Lisez les désaccords** — quand Hermes signale un conflit entre DELPHI et AEGIS, c'est le moment le plus informatif. Ne le sautez pas.

---

*Hermes n'est pas un intermédiaire — c'est le chef d'orchestre. Comprendre son fonctionnement, c'est maîtriser ATLAS.*
