---
name: feature-splitter
description: >
  Découpe un design document en features priorisées et génère les fichiers spec.
  Utilise ce skill après qu'un brainstorm Superpowers a produit un design document
  (docs/plans/*-design.md) pour un nouveau projet. Se déclenche quand le développeur
  dit "découpe en features", "split features", "créer les specs", "prochaine étape
  après le brainstorm", "on découpe", "prépare les features", "rédige les specs",
  ou quand le workflow indique qu'un design document vient d'être validé pour un
  nouveau projet. Ce skill intervient ENTRE le brainstorm Superpowers et /deep-plan.
  Ne PAS déclencher pour un projet existant qui ajoute une seule feature — c'est
  le rôle de deep-plan directement.
---

# Feature Splitter — Découpage et génération des specs

## Pourquoi ce skill existe

Après le brainstorm Superpowers, le design document décrit l'architecture globale
du projet. Mais ce document n'est pas actionnable tel quel — il faut le découper
en features ordonnées que deep-plan pourra planifier une par une.

Sans ce découpage, Superpowers enchaîne directement vers /write-plan et /execute-plan,
ce qui saute la planification approfondie de deep-plan et la validation humaine
par feature.

---

## Prérequis

Avant de lancer ce skill, ces fichiers doivent exister :

1. `planning/vision.md` — la vision produit (généré par Vision Writer)
2. `docs/plans/*-design.md` — le design technique (généré par /brainstorm)

Si l'un des deux manque, signaler au développeur quel fichier est absent et
quelle étape du workflow il doit compléter avant.

---

## Le processus

### Étape 1 — Analyser le design document

Lire le design document et identifier toutes les unités fonctionnelles distinctes.
Une feature est une unité qui :

- A une responsabilité claire et nommable
- Peut être implémentée indépendamment (ou avec des dépendances explicites)
- Peut être testée isolément
- Représente entre 1h et 1 journée de travail pour deep-plan + Superpowers

### Étape 2 — Proposer le découpage au développeur

Présenter la liste des features identifiées dans ce format :

```
╔══════════════════════════════════════════════════════╗
║         DÉCOUPAGE EN FEATURES                       ║
╠══════════════════════════════════════════════════════╣
║ Projet : [nom]                                      ║
║ Basé sur : [nom du design document]                 ║
║ Nombre de features identifiées : [N]                ║
╚══════════════════════════════════════════════════════╝

#  Priorité  Feature                     Dépend de
─────────────────────────────────────────────────────
0  🔴 BASE   Architecture & scaffolding  —
1  🔴 HAUTE  Authentification            #0
2  🔴 HAUTE  [Feature core]              #0
3  🟡 MOY    [Feature]                   #1, #2
4  🟡 MOY    [Feature]                   #2
5  🟢 BASSE  [Feature]                   #3, #4
...

Légende priorité :
🔴 BASE/HAUTE = nécessaire pour le MVP
🟡 MOY = important mais pas bloquant
🟢 BASSE = peut attendre la V2
```

Pour chaque feature, expliquer en une phrase pourquoi elle est à cette priorité
et quelles sont ses dépendances.

### Étape 3 — Attendre la validation

Demander au développeur :
"Ce découpage te convient ? Tu peux :
- Réordonner les priorités
- Fusionner ou séparer des features
- Déplacer des features vers V2 (hors scope actuel)
- Ajouter des features manquantes"

Ne pas avancer tant que le développeur n'a pas validé.

### Étape 4 — Générer les fichiers spec

Pour chaque feature validée, créer un fichier dans `planning/` :

**Nommage** : `planning/XX-nom-feature-spec.md` où XX est le numéro à deux chiffres.

**Contenu de chaque fichier spec** :

```markdown
# Feature XX — [Nom de la feature]

> [Description en une phrase]

## Contexte

[Pourquoi cette feature existe, quel besoin elle adresse.
Référencer la section pertinente du vision.md]

## Dépendances

- Dépend de : Feature #XX [nom] (doit être implémentée avant)
- Bloque : Feature #XX [nom] (attend cette feature)

## Périmètre

### Inclus
- [Ce que cette feature FAIT]
- [Comportement attendu]
- [Cas d'usage couverts]

### Exclus
- [Ce que cette feature NE FAIT PAS]
- [Ce qui sera traité dans une feature ultérieure]

## Critères de validation

- [ ] [Critère testable 1]
- [ ] [Critère testable 2]
- [ ] [Critère testable 3]

## Notes pour deep-plan

[Toute information utile pour la phase de planification approfondie :
questions ouvertes, choix techniques à explorer, risques identifiés]
```

### Étape 5 — Commit et résumé

1. `mkdir -p planning` (si pas déjà fait)
2. Écrire tous les fichiers spec
3. `git add planning/*-spec.md && git commit -m "docs: add feature specs (N features)"`

Puis afficher le résumé :

```
✅ [N] fichiers spec créés dans planning/

Prochaine étape :
  /compact
  /deep-plan @planning/00-architecture-spec.md

Ordre d'implémentation recommandé :
  1. planning/00-architecture-spec.md
  2. planning/01-auth-spec.md
  3. ...
```

---

## Règles de découpage

- **Feature 0 = toujours l'architecture de base.** Scaffolding, config, structure
  de dossiers, setup de test, CI/CD. C'est la fondation sur laquelle tout repose.

- **Respecter le vision.md.** Si une fonctionnalité apparaît dans le design doc
  mais est listée comme "hors scope" dans le vision.md, la proposer en feature
  V2 avec une note explicite.

- **Granularité adaptée.** Une feature "Système d'authentification" peut rester
  une seule feature si elle est simple (email/password). Si elle inclut OAuth,
  2FA, récupération de mot de passe, elle devrait être découpée en sous-features.

- **Pas de features gigantesques.** Si une feature nécessiterait plus d'une
  journée complète avec deep-plan + Superpowers, la découper.

- **Pas de micro-features.** Si une feature prend moins de 30 minutes, la
  fusionner avec une feature adjacente.

---

## Intégration avec le workflow

```
/brainstorm → design doc
       ↓
  ★ FEATURE SPLITTER ★
  Lit le design doc + vision.md
  Propose le découpage
  Attend validation
  Génère les specs
       ↓
  /deep-plan feature par feature
```
