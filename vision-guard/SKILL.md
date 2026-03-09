---
name: vision-guard
description: >
  Vérifie la cohérence du projet avec sa vision originale pour détecter toute dérive.
  Utilise ce skill à trois moments clés : (1) avant de planifier une nouvelle feature
  avec /deep-plan ou /brainstorm, pour vérifier qu'elle est alignée avec la vision,
  (2) après avoir terminé une feature et avant de merge, pour vérifier que
  l'implémentation n'a pas dévié, (3) quand le développeur demande explicitement
  un audit de cohérence, un check de dérive, ou dit des choses comme "est-ce qu'on
  est encore aligné ?", "on dérive pas ?", "vision check", "cohérence", "scope creep",
  "est-ce que ça correspond au projet initial ?". Déclenche aussi quand le développeur
  semble hésiter sur la pertinence d'une feature ("est-ce qu'on a vraiment besoin de
  ça ?", "ça fait pas trop ?"). Ce skill est essentiel pour les projets de plus de
  3-4 features car la dérive s'installe progressivement et silencieusement.
---

# Vision Guard — Détection de dérive projet

## Pourquoi ce skill existe

Au fil des features, un projet dérive naturellement de sa vision originale. Chaque
feature ajoutée peut sembler justifiée individuellement, mais leur accumulation
éloigne le produit de son intention première. C'est le "scope creep silencieux" :
personne ne prend une décision de dévier, mais le résultat final ne ressemble plus
au projet initial.

Ce skill agit comme un garde-fou en comparant systématiquement l'état actuel du
projet avec les documents fondateurs : le fichier de vision et le design document.

---

## Documents de référence

Le skill s'appuie sur ces fichiers (cherche-les dans cet ordre) :

1. **Vision produit** : `planning/vision.md` — le "quoi" et le "pour qui"
   Sert à vérifier : est-ce que cette feature/ce changement est dans le scope ?
   Contient le scope IN, le scope OUT, les utilisateurs, les critères de succès.
   C'est la référence principale pour détecter le scope creep fonctionnel.

2. **Design technique** : `docs/plans/*-design.md` — le "comment"
   Sert à vérifier : est-ce que l'implémentation respecte l'architecture validée ?
   Contient la stack, les composants, les flux de données, la stratégie de test.
   C'est la référence principale pour détecter les dérives techniques
   (changement de stack, de pattern, de structure sans validation).

3. **Feature specs** : `planning/*-spec.md` — le périmètre de chaque feature
4. **CLAUDE.md** : à la racine — les conventions et interdictions

Les deux premiers documents sont complémentaires :
- Le vision.md attrape "on a ajouté une feature qui n'était pas prévue"
- Le design doc attrape "on a changé la stack ou l'architecture sans demander"

Si le vision.md n'existe pas, demande au développeur de le créer (ou de lancer
Vision Writer) avant de continuer. Si le design doc n'existe pas, se baser
uniquement sur le vision.md en signalant que la couverture est partielle.

---

## Les 3 modes d'utilisation

### Mode 1 : Pré-feature (avant de planifier)

Se déclenche avant de lancer `/deep-plan` ou `/brainstorm` sur une nouvelle feature.

Objectif : vérifier que la feature envisagée fait partie de la vision avant d'investir
du temps à la planifier.

Étapes :
1. Lire `planning/vision.md` (scope IN/OUT) et le design document (architecture)
2. Lire la spec de la feature proposée
3. Vérifier deux axes :
   - **Axe fonctionnel** (vs vision.md) : cette feature est-elle dans le scope IN ?
     Contredit-elle un élément du scope OUT ?
   - **Axe technique** (vs design doc) : cette feature est-elle compatible avec
     l'architecture validée ? Nécessite-t-elle un changement d'architecture ?
4. Produire le rapport de cohérence (voir format ci-dessous)
5. Si la feature est hors scope : le signaler clairement et demander au développeur
   s'il veut quand même continuer (c'est son droit — une vision peut évoluer)

### Mode 2 : Post-feature (avant de merge)

Se déclenche après l'exécution d'une feature, avant le merge dans main.

Objectif : vérifier que l'implémentation n'a pas introduit d'éléments hors scope.

Étapes :
1. Lire la vision, le design document et la spec de la feature
2. Analyser les fichiers créés/modifiés dans la branche courante
   (utiliser `git diff main --name-only` et `git diff main --stat`)
3. Vérifier deux axes :
   - **Axe fonctionnel** (vs vision.md + feature spec) : les changements
     correspondent-ils au périmètre de la feature ? Y a-t-il des ajouts
     fonctionnels non prévus dans la spec ?
   - **Axe technique** (vs design doc + CLAUDE.md) : la stack et l'architecture
     sont-elles respectées ? Les conventions de nommage, de structure, de test
     sont-elles suivies ?
4. Identifier tout ajout qui n'était pas dans le plan :
   - Nouvelle dépendance non prévue
   - Fichier hors du scope de la feature
   - Fonctionnalité bonus non demandée
   - Endpoint ou route non prévu
5. Produire le rapport de cohérence

### Mode 3 : Audit global (à la demande)

Se déclenche quand le développeur demande un état des lieux.

Objectif : faire le point sur la cohérence globale du projet.

Étapes :
1. Lire la vision (scope IN/OUT) et le design document (architecture validée)
2. Lister toutes les features implémentées (branches mergées, dossiers existants)
3. Pour chaque feature, évaluer son alignement :
   - **vs vision.md** : la feature sert-elle un objectif du scope IN ?
   - **vs design doc** : la feature respecte-t-elle l'architecture prévue ?
4. Identifier ce qui a été implémenté mais n'était pas dans la vision originale
5. Identifier ce qui était dans la vision mais n'a pas encore été implémenté
6. Détecter les dérives techniques : changements de stack, de patterns, de
   dépendances par rapport au design document
7. Produire le rapport global (voir format ci-dessous)

---

## Format du rapport de cohérence

### Pour les modes 1 et 2 (par feature)

```
╔══════════════════════════════════════════════════════╗
║              VISION GUARD — Rapport                 ║
╠══════════════════════════════════════════════════════╣
║ Feature : [nom de la feature]                       ║
║ Mode    : [Pré-feature / Post-feature]              ║
║ Statut  : [🟢 Alignée / 🟡 Dérive mineure / 🔴 Hors scope] ║
╚══════════════════════════════════════════════════════╝

ALIGNEMENT AVEC LA VISION (vs vision.md)
→ [Expliquer en quoi cette feature sert ou ne sert pas la vision]

ALIGNEMENT TECHNIQUE (vs design doc)
→ [L'implémentation respecte-t-elle l'architecture validée ?]

ÉLÉMENTS DANS LE SCOPE
→ [Lister ce qui est cohérent]

ÉLÉMENTS HORS SCOPE (si applicable)
→ [Lister ce qui dépasse le périmètre prévu]
→ [Pour chaque élément : expliquer pourquoi c'est hors scope]

RECOMMANDATION
→ [Continuer / Ajuster / Reconsidérer]
→ [Si dérive : proposer comment recentrer]
```

### Pour le mode 3 (audit global)

```
╔══════════════════════════════════════════════════════╗
║         VISION GUARD — Audit global                 ║
╠══════════════════════════════════════════════════════╣
║ Projet : [nom]                                      ║
║ Score d'alignement : [X/10]                         ║
╚══════════════════════════════════════════════════════╝

CE QUI EST ALIGNÉ
→ [Feature 1] — 🟢 conforme à la vision
→ [Feature 2] — 🟢 conforme à la vision

CE QUI DÉRIVE
→ [Feature X] — 🟡 la partie [Y] n'était pas prévue dans la vision
→ [Dépendance Z] — 🟡 ajoutée sans justification dans le plan

DÉRIVES TECHNIQUES (vs design doc)
→ [Changement de stack/pattern non validé]
→ [Convention du CLAUDE.md non respectée]

CE QUI MANQUE (prévu mais pas encore fait)
→ [Feature A] — dans la vision, pas encore implémentée
→ [Feature B] — dans la vision, pas encore implémentée

TENDANCE
→ [Le projet est sur les rails / commence à dériver / a significativement dévié]
→ [Raisons principales de la dérive si applicable]

RECOMMANDATIONS
→ [Actions concrètes pour recentrer si nécessaire]
```

---

## Règles de jugement

Le but n'est pas d'être rigide. Les visions évoluent et c'est normal. Le skill doit :

- **Signaler sans bloquer** : toujours laisser le dernier mot au développeur.
  Si une dérive est intentionnelle et assumée, c'est OK. Le rôle du skill est de
  rendre la dérive visible, pas de l'interdire.

- **Distinguer les niveaux** :
  - 🟢 **Alignée** : la feature/le changement sert directement la vision
  - 🟡 **Dérive mineure** : quelques éléments dépassent le scope mais restent
    dans l'esprit du projet (ex: feature bonus petite, dépendance utilitaire)
  - 🔴 **Hors scope** : la feature ou le changement ne correspond pas à la vision
    et ajoute de la complexité sans servir l'objectif principal

- **Être factuel** : citer les passages du vision.md ou du design document qui
  justifient le verdict. Ne pas inventer de règles qui ne sont pas dans les
  documents fondateurs.

- **Penser accumulation** : une seule dérive mineure n'est pas grave. Cinq dérives
  mineures accumulées forment une dérive majeure. L'audit global (mode 3) sert
  justement à détecter cette accumulation.

---

## Quand suggérer de mettre à jour la vision

Si le développeur valide consciemment un changement hors scope, le skill doit
suggérer de mettre à jour `planning/vision.md` pour refléter cette évolution.
Une vision qui ne reflète plus la réalité du projet perd toute utilité de
garde-fou.

Formulation type : "Cette feature dépasse le scope initial. Si c'est intentionnel,
je recommande de mettre à jour planning/vision.md pour intégrer cette direction,
afin que les prochains checks de cohérence soient calibrés sur la vision actuelle."

Pour une dérive technique : "L'implémentation s'écarte de l'architecture validée
dans le design document. Si ce changement est intentionnel, je recommande de mettre
à jour le design doc pour refléter la nouvelle architecture."

---

## Intégration avec le workflow Superpowers + deep-plan

Ce skill ne remplace aucune étape du workflow existant. Il s'intercale aux moments
de transition :

```
Feature spec rédigée
      │
      ▼
  ★ VISION GUARD (mode 1 : pré-feature)
      │
      ▼
  /deep-plan ou /brainstorm
      │
      ▼
  Validation humaine
      │
      ▼
  /write-plan → /execute-plan
      │
      ▼
  ★ VISION GUARD (mode 2 : post-feature)
      │
      ▼
  Merge / PR
```

L'audit global (mode 3) est à lancer manuellement quand le développeur le souhaite,
typiquement toutes les 3-5 features ou à des jalons du projet.
