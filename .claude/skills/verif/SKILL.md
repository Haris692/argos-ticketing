---
name: verif
description: Build, lint, typecheck et tests (rapide, complet ou filtré), puis renvoie un résumé de 15 lignes. À utiliser après chaque intégration.
argument-hint: "[rapide | complet | front | db | fn | <filtre Vitest>]"
context: fork
agent: verificateur
model: haiku
effort: low
background: false
---

Vérifie le projet Argos Ticketing. Portée demandée : « $ARGUMENTS » (vide = rapide).

- **rapide** : typecheck, lint, Vitest.
- **complet** : build, lint, Vitest, `supabase db reset`, `supabase test db`, `deno test supabase/functions`.
- **front** : typecheck, lint, Vitest, build.
- **db** : `supabase db reset`, puis `supabase test db`.
- **fn** : `deno test supabase/functions`.
- **autre valeur** : typecheck, puis Vitest filtré sur « $ARGUMENTS ».

Commandes exactes : `docs/OUTILS.md` §2 (ou `tools/verif.ps1` s'il existe, §3). Sorties complètes dans `.ci/`, seul le résumé remonte.

Rends le format standard (15 lignes au maximum), avec les compteurs exacts et la commande utilisée.
