---
paths:
  - "src/lib/**"
---

# Logique pure (`src/lib/`)

- **Aucun import** de React, de `@supabase/*`, de TanStack Query ni du DOM, sauf dans `src/lib/supabase.ts` (client) et `src/lib/database.types.ts` (généré).
- **`src/lib/time/`** est le moteur de calcul du temps (spec §10), réutilisé tel quel par la future Facturation :
  - paramètres uniquement dans `config.ts` ;
  - stockage en minutes réelles ; regroupement par (ticket, jour, facturable), somme, **puis** arrondi ; jamais d'arrondi entrée par entrée ;
  - le même module sert à l'affichage client, à l'affichage admin et à l'export.
- **Fonctions pures** : mêmes entrées, même sortie ; la date du jour est un paramètre, jamais `new Date()` en interne.
- **Tests** : tout nouvel export a ses tests Vitest dans le même lot (`*.test.ts` à côté du fichier). Liste minimale du moteur de temps : spec §10.
