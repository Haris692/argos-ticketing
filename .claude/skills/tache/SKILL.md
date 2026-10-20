---
name: tache
description: Exécute une tâche de la feuille de route (ex. T2.3) avec le bon agent et le bon modèle, puis la vérifie et la consigne.
argument-hint: "[Tx.y]"
---

Tâche : **$ARGUMENTS**

1. **Trouver la tâche** dans `docs/phases/<Px>.md` (sinon dans `docs/ROADMAP.md`) : objectif, agent, critères, dépendances. Si une dépendance n'est pas terminée, traite-la d'abord ou signale le blocage.
2. **Choisir qui la fait**, selon la table de routage de l'orchestrateur. Si la tâche tient en moins de 15 lignes et que tu as déjà le contexte, fais-la toi-même.
3. **Information manquante sur le code existant** : pose à `Explore` (Haiku) une question précise (« où est défini X ? », « quelles policies existent sur la table Y ? »), jamais « explore le projet ».
4. **Rédiger le brief** selon le gabarit de l'orchestrateur, et lancer l'agent.
5. **Au retour de l'agent** :
   - **OK** : contrôle 1 ou 2 critères clés (`git diff --stat`, un test ciblé). Passe par le `relecteur` ou par `securite` si la règle de revue s'applique.
   - **PARTIEL** : reprends le même agent avec `SendMessage` (il garde son contexte) en indiquant précisément ce qui manque.
   - **ÉCHEC** : applique la règle d'escalade.
6. **Consigner** : `scribe` (STATUS, case ROADMAP, commit).
7. **Informer Haris** en une ligne : « $ARGUMENTS ✓ — <résultat> », plus un point d'attention s'il y en a un.
