# OUTILS — commandes exactes de l'environnement

> Rempli et validé en P1 (T1.1 et T1.3). Une fois validé, c'est **la seule** référence pour builder et tester : on ne redécouvre pas les commandes à chaque session.
> Les valeurs marquées « à confirmer » sont des hypothèses issues de la documentation, pas encore testées sur la machine de Haris.

## 1. Environnement (T1.1)

| Outil | Version | Statut |
|---|---|---|
| Node.js (`node --version`) | — | absent le 2026-10-20 ; LTS récente requise |
| npm (`npm --version`) | — | absent le 2026-10-20 |
| Docker Desktop (`docker info`) | — | à relever ; requis par `supabase start` |
| Supabase CLI (`supabase --version`) | — | absent le 2026-10-20 |
| Deno (`deno --version`) | — | absent le 2026-10-20 ; requis pour `deno test` |
| GitHub CLI (`gh --version`) | — | absent le 2026-10-20 ; optionnel |
| Git | installé ; identité locale au dépôt : Haris | relevé le 2026-10-20 |
| Système et shell | Windows 11, PowerShell 5.1 et Git Bash ; pas de Python | relevé le 2026-10-20 |

## 2. Vérification (à confirmer en T1.3)

| Besoin | Commande | Notes |
|---|---|---|
| Typecheck | `npx tsc -b --noEmit` | à confirmer |
| Lint | `npm run lint` | |
| Tests unitaires | `npm run test -- --run` | Vitest, sans mode watch |
| Tests filtrés | `npx vitest run <filtre>` | |
| Build | `npm run build` | |
| Stack locale | `supabase status`, sinon `supabase start` | Docker doit tourner |
| Rejouer migrations + seed | `supabase db reset` | **jamais** `--linked` |
| Tests RLS | `supabase test db` | pgTAP, `supabase/tests/` |
| Types de la base | `supabase gen types typescript --local > src/lib/database.types.ts` | après chaque migration |
| Edge functions en local | `supabase functions serve` | |
| Tests des edge functions | `deno test supabase/functions` | |

## 3. Script de vérification résumée (T1.3)

- `powershell -NoProfile -ExecutionPolicy Bypass -File tools/verif.ps1 -Scope rapide|complet|front|db|fn` (à écrire).
- Sorties complètes dans `.ci/` (gitignoré), 15 à 25 lignes de résumé au maximum.

## 4. Comptes de test du seed (T1.7)

À remplir : renvoi vers `supabase/seed.sql` (rôle, organisation, email). Les mots de passe de test restent dans le seed, pas dans ce fichier.

## 5. Prod (actions réservées à Haris ou validées par lui)

| Action | Commande |
|---|---|
| Lier le projet | `supabase link --project-ref <ref>` |
| Appliquer les migrations | `supabase db push` |
| Déployer une fonction | `supabase functions deploy <nom>` (`inbound-email` avec `--no-verify-jwt`) |
| Secrets | `supabase secrets set NOM=valeur` |
