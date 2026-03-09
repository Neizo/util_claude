---
name: vision-writer
description: >
  Génère le fichier planning/vision.md par interview structurée. Utilise ce skill
  quand le développeur démarre un nouveau projet, veut créer un fichier de vision,
  dit "nouveau projet", "nouvelle app", "je veux créer...", "j'ai une idée de...",
  "vision.md", "créer la vision", "démarrer un projet", ou toute variante indiquant
  le tout début d'un projet avant même le brainstorming technique. Ce skill
  intervient AVANT /brainstorm de Superpowers — il capture l'intention et le périmètre,
  pas l'architecture. Déclenche aussi quand le développeur veut réécrire ou mettre
  à jour sa vision existante ("mettre à jour la vision", "la vision a changé",
  "revoir le scope du projet"). Ne PAS déclencher pour l'ajout d'une feature
  dans un projet existant (c'est le rôle de deep-plan).
---

# Vision Writer — Interview et génération du vision.md

## Pourquoi ce skill existe

Le fichier `planning/vision.md` est la pierre angulaire du workflow. C'est lui que
Vision Guard utilise comme référence pour détecter les dérives, que Superpowers lit
pendant le brainstorming, et que deep-plan consulte pour chaque feature. S'il est
mal rédigé, vague ou incomplet, toute la chaîne en souffre.

Ce skill garantit que le vision.md couvre toutes les dimensions nécessaires en
posant les questions une par une, sans submerger le développeur.

---

## Philosophie de l'interview

L'objectif est de capturer l'intention du développeur, pas de faire du design
technique. La vision décrit le "quoi" et le "pour qui", jamais le "comment".
Le "comment" viendra ensuite avec /brainstorm puis /deep-plan.

Règles de l'interview :

- **Une question à la fois.** Ne jamais poser deux questions dans le même message.
- **Accepter les réponses courtes.** Si le développeur répond en 3 mots, c'est OK.
  Reformuler et passer à la suite.
- **Proposer des choix quand c'est possible.** Les questions à choix multiples sont
  plus faciles à répondre que les questions ouvertes.
- **Ne pas juger les réponses.** La vision appartient au développeur. Le rôle du
  skill est de la capturer fidèlement, pas de la challenger (ça viendra au brainstorm).
- **Reformuler après chaque réponse.** Un court résumé de ce qu'on a compris avant
  de passer à la question suivante, pour valider au fil de l'eau.
- **Adapter la profondeur.** Si le développeur a déjà une idée très claire, accélérer.
  S'il est flou, creuser davantage.

---

## Les 8 dimensions à couvrir

L'interview suit ces 8 dimensions dans l'ordre. Chaque dimension correspond à une
section du vision.md final. Certaines dimensions peuvent nécessiter 1 à 3 questions
selon la clarté des réponses.

### 1. L'idée en une phrase

Commencer par la question la plus ouverte :
"Décris-moi ton projet en une ou deux phrases — qu'est-ce que c'est ?"

L'objectif est d'obtenir un pitch clair. Si la réponse est vague, reformuler et
demander de préciser. Cette phrase deviendra le titre et le résumé du vision.md.

### 2. Le problème résolu

"Quel problème ce projet résout ? Quelle est la douleur ou le manque aujourd'hui ?"

Comprendre pourquoi ce projet devrait exister. Si le développeur dit "c'est un
projet perso pour apprendre", c'est une réponse valide — noter que le but est
l'apprentissage, pas la résolution d'un problème marché.

### 3. Les utilisateurs cibles

"Qui va utiliser ce produit ? Décris-moi les utilisateurs principaux."

Obtenir au minimum : qui sont-ils, quel est leur contexte, et qu'est-ce qu'ils
essaient d'accomplir. Si c'est un projet perso, l'utilisateur c'est le développeur
lui-même — le noter quand même.

### 4. Les fonctionnalités principales (scope IN)

"Quelles sont les fonctionnalités essentielles sans lesquelles le projet n'a pas
de sens ?"

Lister les features core. Pousser le développeur à prioriser : si tu ne devais en
garder que 3, lesquelles ? C'est le cœur du scope que Vision Guard surveillera.

### 5. Ce qui est hors scope (scope OUT)

"Y a-t-il des choses que les gens pourraient attendre mais que tu ne veux PAS
faire dans ce projet ?"

Cette section est cruciale pour Vision Guard. Les non-objectifs explicites sont
le meilleur outil pour détecter le scope creep. Exemples : "pas de mobile pour
l'instant", "pas de multi-tenant", "pas de marketplace".

Si le développeur n'a pas d'idée, proposer des questions ciblées :
- App mobile nécessaire ou web only ?
- Multi-langue ou français uniquement ?
- Gratuit, payant, freemium ?
- Mono-utilisateur ou multi-utilisateur ?

### 6. Les contraintes connues

"Y a-t-il des contraintes techniques, de budget, de délai ou autres que tu
connais déjà ?"

Exemples : "je veux utiliser Python", "ça doit tourner sur un VPS à 5€/mois",
"c'est pour une démo dans 2 semaines", "je connais bien React mais pas Vue".

Si le développeur n'a pas de contraintes, passer à la suite.

### 7. Les critères de succès

"Comment sauras-tu que le projet est réussi ? Qu'est-ce qui te fera dire
'c'est bon, ça marche' ?"

Peut être technique ("tous les tests passent"), fonctionnel ("un utilisateur
peut créer un compte et faire X"), ou business ("10 utilisateurs actifs").

### 8. L'ambition à long terme (optionnel)

"Si le projet réussit, comment tu le vois évoluer ensuite ? V2, V3..."

Cette section est optionnelle mais utile pour Vision Guard : elle distingue
ce qui est du scope futur (à ne pas implémenter maintenant) de ce qui est
hors scope total (à ne jamais implémenter).

Si le développeur n'a pas de vision long terme, ne pas insister.

---

## Après l'interview : générer le vision.md

Une fois toutes les dimensions couvertes, générer le fichier en utilisant
exactement ce template :

```markdown
# Vision — [Nom du projet]

> [L'idée en une phrase — pitch clair]

## Problème

[Le problème résolu, la douleur, le manque. 2-5 phrases.]

## Utilisateurs cibles

[Qui utilise le produit, dans quel contexte, pour faire quoi.]

## Fonctionnalités principales (scope IN)

- [Feature 1 — courte description]
- [Feature 2 — courte description]
- [Feature 3 — courte description]
- ...

## Hors scope (scope OUT)

- [Ce qu'on ne fait PAS — et pourquoi]
- [Ce qu'on ne fait PAS — et pourquoi]
- ...

## Contraintes

- [Contrainte technique, budget, délai, etc.]
- ...

(Omettre cette section si aucune contrainte identifiée.)

## Critères de succès

- [Critère 1]
- [Critère 2]
- ...

## Vision long terme (optionnel)

[Comment le projet pourrait évoluer en V2, V3.
Ce qui est noté ici n'est PAS dans le scope actuel.]
```

### Règles de rédaction du fichier

- **Fidélité** : utiliser les mots du développeur autant que possible. Ne pas
  reformuler dans un jargon technique si le développeur a parlé simplement.
- **Concision** : chaque section doit être lisible en 10 secondes. Si une section
  dépasse 10 lignes, la résumer.
- **Scope OUT explicite** : c'est la section la plus importante pour Vision Guard.
  Être précis. "Pas de mobile" est mieux que "focus web".
- **Pas de technique** : aucune mention de framework, base de données, ou
  architecture. C'est la vision produit, pas le design technique.

---

## Présenter et valider

Après génération, présenter le vision.md complet au développeur et demander :

"Voici la vision du projet telle que je l'ai comprise. Lis-la et dis-moi si
quelque chose ne correspond pas, manque, ou devrait être reformulé. Ce document
sera la référence pour tout le reste du développement."

Si le développeur valide :
1. Créer le dossier `planning/` s'il n'existe pas : `mkdir -p planning`
2. Sauvegarder dans `planning/vision.md`
3. Commit : `git add planning/vision.md && git commit -m "docs: add project vision"`
4. Indiquer que la prochaine étape est `/brainstorm @planning/vision.md`

Si le développeur veut modifier :
- Appliquer les modifications
- Re-présenter la version corrigée
- Boucler jusqu'à validation

---

## Mode mise à jour

Si un `planning/vision.md` existe déjà, basculer en mode mise à jour :

1. Lire le vision.md existant
2. Demander : "Qu'est-ce qui a changé dans ta vision du projet ?"
3. Poser des questions ciblées uniquement sur les sections concernées
4. Mettre à jour le fichier en préservant les sections inchangées
5. Montrer un diff clair (avant/après) pour que le développeur voit les changements
6. Commit : `git add planning/vision.md && git commit -m "docs: update project vision"`

---

## Intégration avec le workflow

Ce skill s'exécute une seule fois au tout début du projet (phase 01 du workflow).
Il produit le fichier que tous les autres skills utilisent :

```
vision-writer  →  planning/vision.md
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
      /brainstorm   deep-plan   Vision Guard
      (le lit)      (le lit)    (le compare)
```
