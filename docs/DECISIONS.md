# Journal des décisions

> Une ligne par décision. On ne remet pas en cause une décision listée ici sans élément nouveau.
> Pour en changer une, on **ajoute** une ligne qui la remplace (« remplace D-012 ») ; on n'efface jamais.
> Statut : **VALIDÉ** = décidé par Haris · **PROPOSÉ** = choix par défaut de l'équipe, révisable · **À DÉCIDER** = en attente de Haris.

## Décisions

| id | date | décision | statut |
|---|---|---|---|
| D-001 | 2026-10-20 | La spec v2 (`docs/ARGOS-TICKETING-SPEC.md`, octobre 2026) remplace la spec d'août ; c'est la source de vérité | VALIDÉ |
| D-002 | 2026-10-20 | Stack : React 19 + Vite + TS strict + Tailwind 4 + shadcn/ui, HashRouter, TanStack Query, react-hook-form + zod, lucide-react ; Supabase ; GitHub Pages (spec §3, §14.1) | VALIDÉ |
| D-003 | 2026-10-20 | Postmark comme fournisseur email, derrière une interface `EmailProvider` interchangeable (spec §14.2) | VALIDÉ |
| D-004 | 2026-10-20 | Temps comptabilisé = somme par ticket et par jour, arrondie au quart d'heure le plus proche, minimum 15 min (spec §14.3) | VALIDÉ |
| D-005 | 2026-10-20 | Le client voit le temps comptabilisé et les notes publiques, jamais les notes internes (spec §14.4) | VALIDÉ |
| D-006 | 2026-10-20 | Tous les contacts d'une organisation voient tous ses tickets ; les notifications de réponse vont au créateur du ticket uniquement (spec §14.5) | VALIDÉ |
| D-007 | 2026-10-20 | Expéditeur inconnu hors domaines autorisés → quarantaine, jamais de réponse automatique (spec §14.6) | VALIDÉ |
| D-008 | 2026-10-20 | Pas de pièces jointes dans les emails sortants, uniquement un lien vers le portail (spec §14.7) | VALIDÉ |
| D-009 | 2026-10-20 | Commentaire client sur un ticket en attente ou résolu → repasse en cours ; un ticket fermé n'accepte plus de commentaires (spec §14.8) | VALIDÉ |
| D-010 | 2026-10-20 | Organisation Claude Code reprise du projet Brume : un orchestrateur sur Opus et des agents Sonnet ou Haiku selon la table de routage ; skills `/phase`, `/tache`, `/verif`, `/recette`, `/cloture` | VALIDÉ |
| D-011 | 2026-10-20 | Agent `securite` (Opus) à la place du `perf-specialiste` de Brume : revue obligatoire de toute RLS, `SECURITY DEFINER`, Storage et auth d'edge function | PROPOSÉ |
| D-012 | 2026-10-20 | Git avec dépôt GitHub (nécessaire pour Pages et Actions), contrairement à Brume ; `git push`, `git remote` et toute action sur la prod Supabase demandent l'accord de Haris ; jamais de push forcé | PROPOSÉ |
| D-013 | 2026-10-20 | Les phases (P1 à P8, spec §12) jouent le rôle des jalons de Brume ; tâches `Tx.y` ; plan dans `docs/phases/Px.md` ; tag `px` posé après la recette validée par Haris | PROPOSÉ |
| D-014 | 2026-10-20 | Pas de worktree ni de fork : la stack Supabase locale est unique ; un seul agent écrit dans `supabase/migrations/` à la fois | PROPOSÉ |
| D-015 | 2026-10-20 | Identité git du dépôt : « Haris », email du compte, définie localement au dépôt (comme Brume) | PROPOSÉ |

## Idées hors périmètre (à rediscuter plus tard)

| id | date | idée | origine |
|---|---|---|---|
