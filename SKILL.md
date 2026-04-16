---
name: autopilot
description: >
  Impose un workflow structuré en deux phases : PLANIFICATION puis EXÉCUTION.
  Claude doit d'abord produire un plan détaillé, attendre la validation explicite
  de l'utilisateur, puis exécuter le plan complet de manière autonome sans
  commentaires superflus et sans demander d'accord intermédiaire.
  Inclut : filet de sécurité git (commit automatique avant exécution + rollback
  en cas d'échec), checkpoints de progression, parallélisation via subagents
  pour les étapes indépendantes, et apprentissage automatique des erreurs
  dans un fichier lessons.md.
  Utiliser ce skill pour toute tâche non triviale impliquant 3+ étapes :
  développement de features, refactoring, migrations, corrections de bugs
  complexes, mise en place d'architecture, configuration multi-fichiers,
  ou toute demande où l'utilisateur dit "planifie", "fais un plan",
  "prépare avant d'exécuter", ou invoque /autopilot.
---

# Autopilot — Plan → Execute Workflow

## PRINCIPE FONDAMENTAL

Toute tâche suit obligatoirement trois phases séquentielles :
1. **PHASE 1 — PLANIFICATION** : Analyser, produire un plan détaillé, attendre validation.
2. **PHASE 2 — EXÉCUTION** : Implémenter le plan validé intégralement, sans interruption.
3. **PHASE 3 — APPRENTISSAGE** : Capturer les leçons apprises si des erreurs ou adaptations ont eu lieu.

---

## PHASE 0 — PRÉPARATION (automatique, silencieuse)

Avant toute chose, effectuer silencieusement :

1. **Charger les leçons apprises** : Si le fichier `.claude/lessons.md` existe, le lire pour éviter de répéter des erreurs passées. Intégrer ces leçons dans l'analyse du plan.
2. **Vérifier l'état git** :
   - Le répertoire est-il un repo git ?
   - Y a-t-il des changements non commités ? Si oui, les noter dans le plan pour avertir l'utilisateur.

---

## PHASE 1 — PLANIFICATION

### Analyse préalable (silencieuse)

Avant de rédiger le plan, effectuer ces actions en arrière-plan sans les narrer :
- Lire les fichiers pertinents du projet (code source, config, CLAUDE.md)
- Identifier les dépendances et impacts potentiels
- Évaluer la complexité et les risques
- Consulter `.claude/lessons.md` pour les erreurs connues liées au contexte
- Identifier les étapes pouvant être exécutées en parallèle (aucune dépendance mutuelle)

### Format du plan

Produire un plan structuré avec exactement ces sections :

```
## 📋 PLAN — [Titre concis de la tâche]

### Objectif
[1-2 phrases décrivant le résultat attendu]

### Étapes d'implémentation

1. **[Action]** — [Fichier(s) concerné(s)]
   - Détail précis de ce qui sera fait
   - Changements spécifiques (fonctions, classes, config)

2. **[Action]** — [Fichier(s) concerné(s)]
   - Détail précis
   - ...

[Continuer pour chaque étape]

### Groupes parallélisables (si applicable)
- Groupe A (étapes X, Y) — peuvent être exécutées simultanément via subagents
- Groupe B (étapes Z, W) — peuvent être exécutées simultanément via subagents

### Fichiers impactés
- `chemin/fichier1.ext` — [nature de la modification]
- `chemin/fichier2.ext` — [nature de la modification]

### Filet de sécurité git
- ⚠️ Changements non commités détectés : [oui/non — si oui, lister]
- Point de restauration : un commit de sauvegarde sera créé avant l'exécution
- Rollback automatique en cas d'échec critique

### Risques et précautions
- [Risque identifié → Mitigation prévue]

### Leçons appliquées (si applicable)
- [Leçon issue de .claude/lessons.md qui influence ce plan]

### Vérification
- [Comment valider que l'implémentation est correcte]

---
⏳ **En attente de votre validation pour exécuter ce plan.**
Répondez "go", "ok", "valide", ou "exécute" pour lancer l'exécution.
```

### Règles de la Phase 1

- NE PAS écrire de code pendant cette phase
- NE PAS créer, modifier ou supprimer de fichiers
- Utiliser uniquement des outils en lecture seule (Read, Grep, Glob, View)
- Le plan doit être suffisamment détaillé pour être exécuté sans ambiguïté
- Chaque étape doit être atomique et vérifiable
- Si le plan dépasse 15 étapes, le découper en sous-plans numérotés
- Identifier explicitement les étapes parallélisables

---

## TRANSITION — VALIDATION

Attendre une confirmation explicite de l'utilisateur. Accepter comme validation :
- "go", "ok", "oui", "yes", "valide", "validé", "exécute", "lance", "c'est bon", "approved"
- Toute réponse affirmative claire

Si l'utilisateur demande des modifications au plan :
- Mettre à jour le plan
- Présenter la version révisée
- Attendre à nouveau la validation

NE JAMAIS commencer l'exécution sans validation explicite.

---

## PHASE 2 — EXÉCUTION

### 2.1 — Filet de sécurité git (automatique, silencieux)

Immédiatement après validation et AVANT toute modification de fichier :

```bash
# 1. Sauvegarder les changements non commités s'il y en a
git stash --include-untracked -m "autopilot-stash-$(date +%Y%m%d-%H%M%S)" 2>/dev/null

# 2. Créer un commit de sauvegarde sur l'état actuel
git add -A && git commit --allow-empty -m "autopilot-checkpoint: avant exécution — [titre de la tâche]" --no-verify 2>/dev/null

# 3. Stocker le hash pour rollback potentiel
AUTOPILOT_RESTORE_POINT=$(git rev-parse HEAD)
```

Si le projet n'est pas un repo git, ignorer cette étape silencieusement et continuer.

### 2.2 — Exécution séquentielle

Une fois le checkpoint créé, exécuter TOUTES les étapes du plan :

1. **Exécution continue** — Implémenter chaque étape l'une après l'autre sans pause
2. **Zéro commentaire superflu** — Pas de narration, pas d'explication de ce qui va être fait, pas de récapitulatif entre les étapes. Exécuter directement.
3. **Zéro demande d'accord** — Ne pas demander "Voulez-vous que je continue ?" ou "Dois-je procéder ?". Continuer jusqu'à la fin du plan.
4. **Actions silencieuses** — Créer, modifier, supprimer les fichiers sans annoncer chaque action individuellement

### 2.3 — Parallélisation via subagents (si applicable)

Pour les groupes d'étapes identifiés comme parallélisables dans le plan :

- Lancer un subagent par étape indépendante
- Chaque subagent reçoit : le contexte minimal nécessaire + l'instruction précise de son étape
- Attendre la complétion de tous les subagents du groupe avant de passer au groupe suivant
- Si les subagents ne sont pas disponibles, exécuter les étapes séquentiellement

Format de délégation subagent :
```
Subagent [nom-descriptif] :
- Tâche : [description exacte de l'étape]
- Fichiers : [fichiers à modifier]
- Contraintes : [règles à respecter]
- Vérification : [comment confirmer le succès]
```

### 2.4 — Checkpoints de progression

Après chaque étape majeure (ou groupe d'étapes parallèles) :

```bash
git add -A && git commit -m "autopilot-step-[N]: [description courte]" --no-verify 2>/dev/null
```

Cela permet :
- De visualiser la progression via `git log`
- De revenir à n'importe quelle étape intermédiaire
- De ne pas perdre le travail en cas d'interruption

Si le projet n'est pas un repo git, ignorer les checkpoints silencieusement.

### 2.5 — Gestion d'erreur autonome

Si une erreur survient pendant l'exécution :

**Erreur mineure** (n'affecte pas les étapes suivantes) :
- Corriger silencieusement et continuer
- Noter l'adaptation pour le rapport final

**Erreur majeure** (bloque la suite du plan) :
1. Tenter 2 approches alternatives maximum
2. Si résolue → noter l'adaptation, continuer
3. Si non résolue → exécuter le rollback :
   ```bash
   git reset --hard $AUTOPILOT_RESTORE_POINT
   git stash pop 2>/dev/null
   ```
4. Informer l'utilisateur avec :
   - L'étape qui a échoué
   - L'erreur rencontrée
   - Les approches tentées
   - L'état du projet (restauré au point de départ)

### Ce qui est interdit pendant l'exécution

- ❌ "Je vais maintenant..." / "Passons à l'étape suivante..."
- ❌ "Voulez-vous que je continue ?"
- ❌ "Est-ce que ça vous convient ?"
- ❌ Résumer ce qui vient d'être fait entre chaque étape
- ❌ Demander confirmation pour des décisions déjà couvertes par le plan
- ❌ Tout commentaire qui ne soit pas une action concrète

### Ce qui est autorisé pendant l'exécution

- ✅ Messages d'erreur critiques nécessitant une décision non couverte par le plan
- ✅ Brève indication si une étape a dû être adaptée (1 ligne max)
- ✅ Exécution de tests/vérifications prévus dans le plan

### Rapport de fin d'exécution

Une fois TOUTES les étapes terminées, fournir un rapport concis :

```
## ✅ EXÉCUTION TERMINÉE

### Résumé
- [X] étapes complétées sur [Y]
- Fichiers créés : [liste]
- Fichiers modifiés : [liste]
- Fichiers supprimés : [liste]

### Checkpoints git
- [N] commits de progression créés
- Point de restauration initial : [hash court]
- Pour annuler toute l'opération : `git reset --hard [hash]`

### Adaptations par rapport au plan
- [Uniquement si des ajustements ont été nécessaires]

### Vérification
- [Résultats des tests/vérifications effectués]

### Prochaines actions suggérées (optionnel)
- [Si pertinent]
```

---

## PHASE 3 — APPRENTISSAGE (automatique)

Après le rapport final, si des erreurs ou adaptations significatives ont eu lieu pendant l'exécution :

1. **Mettre à jour `.claude/lessons.md`** en ajoutant les nouvelles leçons :

```markdown
## [Date] — [Titre de la tâche]

### Problème rencontré
[Description concise du problème]

### Cause
[Pourquoi c'est arrivé]

### Solution appliquée
[Comment ça a été résolu]

### Règle à suivre
[Instruction claire pour éviter ce problème à l'avenir]
```

2. **Créer le fichier** `.claude/lessons.md` s'il n'existe pas encore
3. **Ne pas dupliquer** les leçons déjà présentes
4. Ne consigner que les leçons **actionables** — pas les événements triviaux

Cette phase est silencieuse. Ne pas annoncer la mise à jour du fichier sauf si c'est la première création.

---

## NOTES D'UTILISATION

- Ce skill s'applique à toute tâche invoquée via `/autopilot [description de la tâche]`
- Il peut aussi être déclenché automatiquement pour des tâches complexes (3+ fichiers, refactoring, migration)
- Pour des tâches simples (correction d'un bug trivial, modification d'une ligne), ce workflow n'est pas nécessaire — utiliser le jugement
- Compatible avec `--dangerously-skip-permissions` pour une exécution encore plus fluide
- Les checkpoints git permettent de revenir à n'importe quel point via `git log --oneline --grep="autopilot"`
- Le fichier `.claude/lessons.md` s'enrichit au fil du temps et rend Claude de plus en plus fiable sur le projet
- La parallélisation via subagents est optionnelle et ne s'active que si des étapes sont réellement indépendantes
