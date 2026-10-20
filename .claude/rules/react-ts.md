---
paths:
  - "src/**/*.ts"
  - "src/**/*.tsx"
---

# Règles TypeScript / React (chargées quand on touche du front)

- **Organisation** : une feature par dossier `src/features/<domaine>/` (composants, hooks, schémas zod) ; un composant exporté par fichier, nommé comme le fichier.
- **Typage** : strict, pas de `any`, pas de `as` pour masquer une erreur. Types de la base depuis `src/lib/database.types.ts` (généré : ne jamais l'éditer à la main).
- **Données** : TanStack Query pour toute lecture et écriture Supabase, avec des clés de cache centralisées par feature ; invalider ce qui change après une mutation. Pas d'appel Supabase dans un composant de présentation.
- **Formulaires** : react-hook-form + zod ; les messages d'erreur en français.
- **Routing** : HashRouter ; les routes et les gardes par rôle vivent dans `src/app/`.
- **Rendu** : contenu utilisateur et emails entrants toujours en texte ; **jamais** `dangerouslySetInnerHTML`.
- **Secrets** : seules `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` et `VITE_SUPPORT_ADDRESS` existent côté front.
- **Styles** : classes Tailwind et tokens CSS ; aucune couleur en dur (spec §11).
- **Dates** : affichage et saisie en Europe/Paris ; jamais de calcul de date « à la main » dans un composant (helpers dans `src/lib/`).
