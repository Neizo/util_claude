# CLAUDE.md — Règles du projet

## Workflow de développement

Ce projet utilise deux plugins complémentaires : **deep-plan** et **Superpowers**.
Ils ne doivent pas se chevaucher. Voici les règles strictes à suivre.

### Règle 1 : Choisir le bon outil selon la complexité

Avant de lancer le brainstorming Superpowers ou deep-plan, évalue la complexité :

**Utiliser directement Superpowers (brainstorm → plan → execute) quand :**
- Bug fix ou correction simple
- Feature isolée dans 1-3 fichiers
- Refactoring localisé
- Les exigences sont déjà claires
- Pas besoin de recherche sur les bonnes pratiques

**Utiliser /deep-plan PUIS Superpowers quand :**
- Feature majeure touchant plusieurs modules
- Exigences vagues ou incomplètes
- Besoin de recherche web sur les meilleures pratiques actuelles
- Architecture nouvelle ou choix technologique structurant
- Intégration avec des APIs/services externes peu familiers
- Le développeur dit explicitement "deep-plan" ou "planifie en profondeur"

### Règle 2 : Quand /deep-plan est utilisé, NE PAS lancer le brainstorming Superpowers

Si le développeur lance `/deep-plan @fichier-spec.md`, le processus de planification
est entièrement géré par deep-plan. Ne pas déclencher le brainstorming de Superpowers
en parallèle ou après — c'est redondant.

### Règle 3 : Le handoff deep-plan → Superpowers

Une fois que /deep-plan a produit ses livrables (plan, sections, TDD stubs), 
le workflow Superpowers prend le relais UNIQUEMENT pour l'exécution :

1. **Passer les sections de deep-plan à Superpowers write-plan**
   Les fichiers `sections/section-*.md` produits par deep-plan servent de base.
   Utiliser `/write-plan` en pointant vers le plan généré par deep-plan 
   (`planning/claude-plan.md`) pour créer les tâches granulaires de 2-5 min.

2. **Exécuter avec /execute-plan**
   Les sous-agents Superpowers implémentent chaque tâche avec TDD obligatoire.

3. **Code review et finalisation**
   Le skill requesting-code-review valide chaque tâche.
   Le skill finishing-a-development-branch gère la PR/merge.

### Règle 4 : En cas de doute, demander

Si tu hésites entre les deux workflows, demande au développeur :
"Cette feature est-elle assez complexe pour justifier une planification approfondie 
avec /deep-plan (recherche + interview + review multi-LLM), ou on part sur un 
brainstorming rapide avec Superpowers ?"

### Règle 5 : Donner des informations
Quand tu utilises un skill, annonce-le explicitement : 'J'utilise [nom du skill] pour [raison]

## Commandes de référence

| Commande | Quand l'utiliser |
|---|---|
| `/deep-plan @spec.md` | Lancer la planification approfondie |
| `/brainstorm` | Brainstorming rapide (features simples) |
| `/write-plan` | Transformer un design validé en tâches |
| `/execute-plan` | Exécuter les tâches via sous-agents |
| `/compact` | Toujours faire avant /deep-plan (gros contexte) |

## RÈGLE CRITIQUE — Arrêt après brainstorm

Quand /brainstorm produit un design document pour un NOUVEAU PROJET 
(pas une feature isolée), le workflow DOIT s'arrêter ici.

NE PAS enchaîner avec /write-plan ou /execute-plan.

À la place, dire au développeur :
"Le design document est prêt. La prochaine étape est de découper 
le projet en features et de rédiger un fichier spec par feature 
dans planning/ possibilité d'utiliser le skill /feature-splitter. Ensuite on utilisera /deep-plan sur chaque feature 
avant d'implémenter."

Cette règle ne s'applique PAS aux features simples lancées 
directement avec /brainstorm dans un projet existant — dans ce cas, 
le pipeline normal Superpowers continue.