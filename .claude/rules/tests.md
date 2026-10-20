---
paths:
  - "src/**/*.test.ts"
  - "src/**/*.test.tsx"
  - "supabase/tests/**"
  - "supabase/functions/**/*_test.ts"
  - "supabase/functions/**/*.test.ts"
---

# Règles des tests

- **Nommage** : une description qui dit la règle (« deux entrées même ticket même jour sont sommées avant arrondi »). Un seul concept par test.
- **Déterminisme** : dates fixes, fuseau Europe/Paris explicite, aucun appel réseau, aucun ordre implicite entre tests.
- **Cas limites obligatoires** : zéro, bornes (fin et début de mois), listes vides, entrées invalides, caractères spéciaux (`;`, `"`, retour à la ligne).
- **pgTAP** : chaque fichier commence par `begin;` et finit par `rollback;` ; on teste en se mettant dans la peau d'un rôle (`authenticated` + claims, `anon`) ; on teste **ce qui est interdit** autant que ce qui est permis. Un contact LSM, un contact EUR, un contact désactivé et l'admin au minimum.
- **Données de test** : petites fixtures inline ; pgTAP s'appuie sur des données créées dans le test, pas sur le seed, sauf mention contraire.
- **Test instable** : c'est un bug. On le corrige, ou on le marque `skip` avec la raison et la tâche, et on le signale dans STATUS.
