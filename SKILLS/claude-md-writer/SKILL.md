---
name: claude-md-writer
description: >
  Génère ou met à jour le CLAUDE.md du projet à partir des décisions prises
  pendant la planification. Utilise ce skill après que deep-plan a produit
  le plan de la Feature 0 (architecture de base), car c'est à ce moment que
  la stack, les conventions et la structure sont connues. Se déclenche quand
  le développeur dit "crée le CLAUDE.md", "génère le CLAUDE.md", "écris les
  règles du projet", "configure Claude pour le projet", ou quand le plan de
  la Feature 0 vient d'être validé et qu'aucun CLAUDE.md n'existe encore.
  Déclenche aussi pour "mets à jour le CLAUDE.md", "ajoute une règle au
  CLAUDE.md", "la convention a changé". Ne PAS déclencher si un CLAUDE.md
  complet existe déjà et que le développeur ne demande pas de modification.
---

# CLAUDE.md Writer — Génération du contrat de projet

## Pourquoi ce skill existe

Le CLAUDE.md est le fichier que Claude Code lit au démarrage de chaque session.
Il définit les règles, les conventions et les interdictions du projet. Sans lui,
Claude Code travaille à l'aveugle — il ne connaît pas la stack, les conventions
de nommage, ni les règles de workflow entre les skills.

Ce skill automatise la rédaction en extrayant les informations des documents
déjà produits par le workflow (vision.md, design doc, plan deep-plan), au lieu
de forcer le développeur à tout réécrire manuellement.

---

## Prérequis

Ce skill a besoin d'au moins un de ces fichiers pour travailler :

1. `planning/vision.md` — vision produit (scope IN/OUT)
2. `docs/plans/*-design.md` — design technique (stack, architecture)
3. `planning/claude-plan.md` ou `planning/00-*-spec.md` — plan Feature 0

Plus il y a de fichiers disponibles, plus le CLAUDE.md sera complet.
Si aucun fichier n'existe, signaler au développeur qu'il doit d'abord
passer par Vision Writer → /brainstorm → deep-plan avant de générer
le CLAUDE.md.

---

## Le processus

### Étape 1 — Collecter les informations

Lire dans l'ordre :

1. **vision.md** → extraire : nom du projet, description, scope IN/OUT
2. **design doc** → extraire : stack technique, architecture, patterns,
   structure de dossiers, stratégie de test, dépendances principales
3. **plan Feature 0** → extraire : conventions de nommage, commandes
   de build/test/lint, configuration choisie (ORM, framework, etc.)
4. **CLAUDE.md existant** (si mode mise à jour) → préserver les règles
   manuelles ajoutées par le développeur

### Étape 2 — Générer le CLAUDE.md

Utiliser exactement ce template, en remplissant chaque section avec les
informations extraites :

```markdown
# CLAUDE.md — [Nom du projet]

> [Description du projet en une phrase, extraite du vision.md]

## Stack technique

- **Langage** : [ex: TypeScript 5.x]
- **Framework** : [ex: Next.js 15]
- **Base de données** : [ex: PostgreSQL + Prisma]
- **Tests** : [ex: Vitest + Testing Library]
- **Linter/Formatter** : [ex: ESLint + Prettier]
- **Déploiement** : [ex: Vercel / Docker / VPS]

## Structure du projet

[Arborescence des dossiers principaux, extraite du design doc]

```
src/
├── features/       ← logique métier par feature
├── shared/         ← utilitaires partagés
├── config/         ← configuration
└── ...
```

## Commandes

```bash
npm run dev          # Lancer en développement
npm run build        # Build production
npm run test         # Lancer les tests
npm run lint         # Linter
npm run format       # Formatter
```

## Conventions

### Nommage
- Fichiers : [ex: kebab-case]
- Composants : [ex: PascalCase]
- Variables/fonctions : [ex: camelCase]
- Types/Interfaces : [ex: PascalCase, préfixe I pour interfaces si convention]

### Git
- Commits : [ex: Conventional Commits (feat:, fix:, docs:, ...)]
- Branches : [ex: feature/XX-nom, fix/XX-nom]

### Tests
- Dossier : [ex: __tests__/ à côté du fichier testé]
- Nommage : [ex: nom-du-fichier.test.ts]
- Couverture minimale : [ex: pas de minimum imposé / 80%]

## Workflow de développement

### Quand utiliser chaque outil

| Situation | Outil |
|---|---|
| Nouveau projet → vision | Vision Writer |
| Vision → architecture | /brainstorm (Superpowers) |
| Architecture → features | Feature Splitter |
| Feature complexe → plan | /deep-plan |
| Feature simple → plan | /brainstorm (Superpowers) |
| Plan validé → code | /write-plan + /execute-plan (Superpowers) |
| Bug fix simple | Superpowers directement |

### Règles de handoff

- Après /brainstorm pour un NOUVEAU PROJET : **STOP**. Ne pas enchaîner
  avec /write-plan. Le développeur doit d'abord découper en features.
- Après /deep-plan : attendre la validation humaine explicite avant
  de passer à /write-plan.
- Pendant /execute-plan : tout écart du plan → arrêt et question.

### Vision Guard

Avant chaque /deep-plan : vérifier l'alignement avec planning/vision.md
Avant chaque merge : vérifier les dérives dans le diff
Toutes les 3-5 features : audit global de cohérence

## Interdictions

- ❌ Pas de code sans plan validé par le développeur
- ❌ Pas de nouvelle dépendance sans justification
- ❌ Pas de modification hors scope de la feature en cours
- ❌ Pas de brainstorm Superpowers si deep-plan est déjà actif
- ❌ Pas d'enchaînement automatique brainstorm → execute sur un nouveau projet

## Scope du projet

### Ce qu'on fait (scope IN)
[Extrait de vision.md, section "Fonctionnalités principales"]

### Ce qu'on ne fait PAS (scope OUT)
[Extrait de vision.md, section "Hors scope"]
```

### Règles de rédaction

- **Extraire, ne pas inventer.** Chaque information doit venir d'un document
  existant. Si une information manque (ex: pas de convention de nommage dans
  le design doc), poser la question au développeur plutôt qu'inventer.

- **Questions si trous.** Après la collecte, si des sections restent vides,
  poser les questions manquantes au développeur, une par une :
  "Le design doc ne précise pas la convention de nommage des fichiers.
  Tu préfères kebab-case, camelCase, ou autre ?"

- **Garder le scope IN/OUT.** Ces sections sont cruciales pour Vision Guard.
  Les copier fidèlement depuis le vision.md.

- **La section Workflow est standard.** Elle ne change pas d'un projet à l'autre
  (sauf si le développeur n'utilise pas l'un des skills). La garder telle quelle.

### Étape 3 — Présenter et valider

Afficher le CLAUDE.md complet au développeur et demander :

"Voici le CLAUDE.md généré à partir de ta vision, du design document
et du plan d'architecture. Relis-le et dis-moi si quelque chose
manque ou doit être modifié. Ce fichier sera lu par Claude Code à
chaque session."

Boucler jusqu'à validation.

### Étape 4 — Sauvegarder

1. Écrire le fichier à la racine du projet : `CLAUDE.md`
2. `git add CLAUDE.md && git commit -m "docs: add CLAUDE.md project rules"`

---

## Mode mise à jour

Si un CLAUDE.md existe déjà :

1. Lire le fichier existant
2. Demander : "Qu'est-ce qui a changé ?" ou identifier automatiquement
   les changements si le développeur signale une nouvelle convention
3. Modifier uniquement les sections concernées
4. Préserver toutes les règles manuelles ajoutées par le développeur
   (elles sont souvent en fin de fichier ou dans des sections custom)
5. Montrer le diff avant/après
6. `git add CLAUDE.md && git commit -m "docs: update CLAUDE.md"`

---

## Intégration avec le workflow

```
/deep-plan Feature 0 → plan validé
            ↓
   ★ CLAUDE.MD WRITER ★
   Lit : vision.md + design doc + plan
   Extrait les conventions et la stack
   Pose les questions manquantes
   Génère le CLAUDE.md
   Attend validation
            ↓
   Validation humaine du plan Feature 0
            ↓
   /write-plan + /execute-plan
```
