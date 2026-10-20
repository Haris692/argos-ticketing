---
name: phase
description: Lance ou reprend une phase de la feuille de route (plan de l'architecte, questions à Haris, exécution des tâches, clôture et arrêt pour recette).
argument-hint: "[P1..P8]"
---

Phase demandée : **$ARGUMENTS**

1. **Contexte.** Lis `docs/STATUS.md`, puis seulement la section de la phase `$ARGUMENTS` dans `docs/ROADMAP.md` (`grep -n "^## $ARGUMENTS" docs/ROADMAP.md`, puis lecture ciblée).
2. **Prérequis.** Vérifie que la phase précédente est validée par Haris (STATUS : « Recette validée »). Sinon, arrête-toi et demande-lui s'il veut enchaîner quand même. Vérifie aussi les prérequis de la spec §16 qui concernent cette phase.
3. **Plan.** Si `docs/phases/$ARGUMENTS.md` n'existe pas, délègue sa rédaction à `architecte`, selon le gabarit de `docs/phases/README.md`.
   - Le brief contient l'objectif de la phase, les sections de la spec à lire, l'état actuel (extrait de STATUS, dont les questions ouvertes) et les contraintes : 150 lignes au maximum, tâches exécutables par Sonnet ou Haiku, migrations ordonnées, parallélisme explicite.
4. **Relecture du plan.** C'est le seul long document que tu lis en entier. Vérifie que chaque tâche a un agent, un modèle et des critères vérifiables, que les dépendances tiennent, que les tâches en parallèle ne partagent aucun fichier, et que chaque nouvelle table a son test pgTAP dans le même lot.
5. **Présenter le plan à Haris** en 10 lignes au maximum : fichiers, migrations, tests prévus. Pose en même temps les questions bloquantes (AskUserQuestion si l'outil existe), recommandation en premier. Pendant ce temps, lance les tâches qui ne dépendent pas de ces réponses.
6. **Exécution.** Les tâches suivent l'ordre du plan, avec le protocole `/tache`. Lance ensemble ce qui est marqué parallèle (3 agents de code au maximum, un seul sur les migrations).
7. **Clôture, quand tout est coché.**
   a. `/verif complet`.
   b. `relecteur` sur le diff de la phase (base : le tag de la phase précédente, ou le premier commit).
   c. `securite` sur le diff de la phase pour P1, P3, P4, P5, P6 et P8.
   d. Les points BLOQUANT sont corrigés par l'agent de dev concerné, puis on refait `/verif`.
   e. `/recette $ARGUMENTS` produit la checklist pour Haris.
   f. `scribe` met à jour STATUS, ROADMAP et DECISIONS (dont les décisions du plan) et fait le commit. Le tag `p1`, `p2`… est posé **après** la validation de Haris.
8. **Bilan à Haris** en 10 lignes au maximum, puis **arrêt** : on attend sa validation explicite avant la phase suivante. Recommande `/clear` avant de l'enchaîner.
