# Argos Ticketing — instructions pour Claude Code

La spécification complète est dans `docs/ARGOS-TICKETING-SPEC.md`. C'est la source de vérité : relis la section concernée avant chaque phase.

## Façon de travailler

- **Une phase à la fois** (spec, section 12). Au début d'une phase, présente un plan court : fichiers, migrations, tests prévus.
- **À la fin de chaque phase, arrête-toi** : résume ce qui a été fait, liste ce que Haris doit tester ou configurer, et attends sa validation explicite avant de passer à la suivante.
- Point ambigu ou contradiction avec la spec → pose la question, n'invente pas.
- Les décisions de la section 14 sont tranchées : ne les change pas de ta propre initiative.
- Ce qui est en section 15 (hors périmètre) ne s'implémente pas, même « pour préparer la suite ».
- Commits atomiques, format Conventional Commits (`feat:`, `fix:`, `test:`, `chore:`…).

## Conventions

- TypeScript strict, pas de `any`.
- Interface entièrement en français.
- Toute modification de schéma = **nouvelle migration**. Ne jamais éditer une migration déjà appliquée.
- Après chaque migration : `supabase gen types typescript --local > src/lib/database.types.ts`.
- Logique métier pure dans `src/lib/`, sans dépendance à React ni à Supabase, couverte par Vitest.
- Composants shadcn/ui, couleurs uniquement via les tokens CSS (spec, section 11). Toujours vérifier en clair **et** en sombre.

## Sécurité — non négociable

- Jamais de clé secrète Supabase (service_role / secret) côté front ni dans le repo. Les variables `VITE_*` sont publiques.
- Toute nouvelle table : RLS activée + policies + test pgTAP dans le même lot.
- Fonctions `SECURITY DEFINER` : `set search_path = ''`, `execute` révoqué pour `anon`.
- Aucun `dangerouslySetInnerHTML`. Contenu utilisateur et emails entrants rendus en texte brut.
- Secrets via `supabase secrets set` et GitHub Secrets uniquement.

## Commandes

```bash
npm run dev            # front en local
npm run build
npm run lint
npm run test           # Vitest
supabase start         # stack Supabase locale
supabase db reset      # rejoue migrations + seed
supabase test db       # tests pgTAP
supabase functions serve
deno test supabase/functions   # tests des edge functions
```

## Avant de déclarer une phase terminée

- [ ] `npm run build` et `npm run lint` passent
- [ ] `npm run test` et `supabase test db` sont verts
- [ ] Vérification visuelle en clair et en sombre, et à 375 px de large
- [ ] Résumé de la phase + liste des actions de Haris envoyés, puis arrêt
