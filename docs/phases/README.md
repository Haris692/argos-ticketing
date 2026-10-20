# Plans de phase

Ce dossier contient un fichier `Px.md` par phase, rédigé par l'agent `architecte` au lancement de la phase (`/phase Px`), et un fichier `Px-recette.md` écrit en fin de phase (`/recette Px`).

Gabarit d'un plan (court : 150 lignes maximum) :

1. **Objectif et périmètre** : ce qui est dedans, ce qui est explicitement dehors (phases suivantes, spec §15).
2. **Décisions techniques** : chacune reportée en une ligne dans `docs/DECISIONS.md`.
3. **Schéma et interfaces** : tables et colonnes, signatures SQL (fonctions, triggers, policies nommées), signatures TypeScript ; pas d'implémentation.
4. **Migrations** : liste ordonnée, une par tâche.
5. **Fichiers à créer ou modifier**, par tâche.
6. **Découpage des tâches** : ID, agent, modèle, dépendances, critères d'acceptation vérifiables, tests attendus (Vitest, pgTAP, Deno).
7. **Parallélisme** : quelles tâches peuvent tourner en même temps, sans toucher les mêmes fichiers (jamais deux tâches sur `supabase/migrations/`).
8. **Risques** et plan B.
9. **Questions pour Haris** : uniquement ce qui bloque vraiment.
