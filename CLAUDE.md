# ARGOS TICKETING : instructions du projet

> Ce fichier est chargé par **tous** les agents à chaque démarrage : il doit rester court. Les détails vivent dans `docs/` et dans `.claude/rules/` (règles chargées seulement quand on touche les fichiers concernés).

## Le projet

Outil de ticketing de Haris, consultant IT / cybersécurité indépendant (Lyon), pour ses clients freelance (au lancement : **LSM Logistics** `LSM` et **Eurofret** `EUR`).
Un seul admin (Haris) ; les contacts clients ouvrent et suivent leurs tickets par **portail web** ou **par email** ; chaque ticket porte le **temps passé**, visible par le client et exportable par mois.
Stack : React 19 + Vite + TS strict + Tailwind 4 + shadcn/ui, Supabase (Postgres, RLS, Storage, Edge Functions), Postmark, GitHub Pages. Premier module de la suite **Argos** (la Facturation viendra plus tard).

## Carte des documents

Toujours lire **par section**, jamais un document en entier : `grep -n "^## " <fichier>`, puis lecture avec offset/limit.

| Besoin | Fichier |
|---|---|
| État courant, prochaine tâche | `docs/STATUS.md` |
| **Source de vérité** : métier, données, RLS, emails, écrans, design | `docs/ARGOS-TICKETING-SPEC.md` (§n) |
| Phases et tâches | `docs/ROADMAP.md`, puis `docs/phases/Px.md` |
| Décisions déjà prises (ne pas re-débattre) | `docs/DECISIONS.md` |
| Commandes de build, de tests et de vérification | `docs/OUTILS.md` |

## Arborescence

- `src/app` (routes, layouts, gardes par rôle), `src/features/<domaine>`, `src/components/ui` (shadcn), `src/styles` (tokens).
- `src/lib/` : logique pure, sans React ni Supabase (dont `src/lib/time/`, moteur de temps réutilisé par la future Facturation).
- `supabase/migrations`, `supabase/functions/{_shared,send-email,inbound-email,manage-contact}`, `supabase/tests` (pgTAP), `supabase/seed.sql`.
- `tools/` : scripts de vérification qui résument leurs sorties. `.ci/` : sorties et logs (gitignoré).

## Règles communes à tous les agents

1. **Économiser les tokens.**
   - Grep et Glob avant Read ; offset/limit au-delà de 200 lignes.
   - Ne jamais lire `node_modules/`, `dist/`, `src/lib/database.types.ts` en entier, ni un log complet.
   - Les sorties longues partent dans `.ci/` ; on n'en affiche qu'un résumé (`tools/`, `tail`, `grep` ciblé).
2. **Rester dans le périmètre.** La tâche demandée, rien de plus, sans refactor opportuniste. Ce qui est hors périmètre (spec §15) ne s'implémente pas, même « pour préparer la suite ». Une idée devient une ligne dans `docs/DECISIONS.md`, section « Idées ».
3. **Spec.** Un point ambigu ou contradictoire : on pose la question, on n'invente pas. Les décisions de la spec §14 et de DECISIONS (VALIDÉ) ne se changent pas de notre propre initiative. La spec n'est modifiée qu'avec l'accord de Haris.
4. **Code.**
   - TypeScript strict, pas de `any`. Identifiants et commentaires en anglais ; **interface entièrement en français**.
   - Logique métier pure dans `src/lib/`, couverte par Vitest.
   - Toute modification de schéma = **nouvelle migration** ; on n'édite jamais une migration appliquée. Après chaque migration : régénérer `src/lib/database.types.ts` (`docs/OUTILS.md`).
   - Couleurs uniquement via les tokens CSS (spec §11) ; toujours vérifier en clair **et** en sombre, et à 375 px.
5. **Sécurité, non négociable.**
   - Jamais de clé secrète Supabase (service_role / secret) côté front ni dans le repo. Les variables `VITE_*` sont publiques.
   - Toute nouvelle table : RLS activée + policies + test pgTAP **dans le même lot**.
   - Fonctions `SECURITY DEFINER` : `set search_path = ''`, noms qualifiés, `execute` révoqué pour `anon`.
   - Aucun `dangerouslySetInnerHTML`. Contenu utilisateur et emails entrants rendus en texte brut.
   - Secrets via `supabase secrets set` et GitHub Secrets uniquement.
6. **Langue.** Docs et messages à Haris en français, concis.
7. **Accord de Haris obligatoire** pour : un package hors de la stack (spec §3), toute action sur la **prod Supabase** (`link`, `db push`, `functions deploy`, `secrets set`), `git push`, et tout ce qui est irréversible. Jamais de `supabase db reset --linked`.

## Definition of Done

Une tâche est finie quand :
- `npm run build`, `npm run lint` et le typecheck passent sans nouvelle erreur ni nouveau warning.
- Les tests de la zone touchée passent (Vitest, `supabase test db`, `deno test`), et toute nouvelle logique pure ou règle RLS a ses tests.
- Chaque critère d'acceptation a été **vérifié**, pas supposé.
- `docs/STATUS.md` et la case de `docs/ROADMAP.md` sont à jour, avec un commit atomique `type(zone): message` (Conventional Commits).

## Format de retour d'un sous-agent

Obligatoire, 15 lignes au maximum :

```
RÉSULTAT : OK | PARTIEL | ÉCHEC
FAIT : <1 à 3 lignes>
FICHIERS : <chemins créés ou modifiés>
VÉRIF : <build ok/ko · lint · tests x/y · commande utilisée>
OUVERT : <reste à faire, risques, questions, vérifications manuelles — ou « rien »>
```

Pas de code ni de log dans le retour : le code est sur le disque.

# Compact instructions

Lors d'une compaction, garder : la tâche en cours et ses critères, les fichiers modifiés, les résultats de vérification, les décisions prises et les questions ouvertes. Laisser tomber : le contenu des fichiers lus et les sorties de commandes.
