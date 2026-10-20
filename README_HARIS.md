# Kit « Argos Ticketing » pour Claude Code : mode d'emploi

Ce kit reprend l'organisation du projet Brume : Claude Code développe Argos de façon organisée et économe en tokens, phase par phase, et s'arrête à la fin de chaque phase pour ta recette.

## Ce qu'il y a dedans

| Élément | Rôle |
|---|---|
| `CLAUDE.md` | Les règles communes, lues par tous les agents (volontairement court) |
| `.claude/agents/` | L'équipe : 1 orchestrateur et 9 agents spécialisés |
| `.claude/skills/` | Les commandes `/phase`, `/tache`, `/verif`, `/recette`, `/cloture` |
| `.claude/rules/` | Des règles chargées **seulement** quand on touche certains fichiers (React, logique pure, SQL, edge functions, tests, design, docs) |
| `.claude/settings.json` | Orchestrateur par défaut, permissions, garde-fous (prod Supabase et `git push` sur demande) |
| `docs/` | Spec (source de vérité), feuille de route, statut, décisions, outils, plans de phase |
| `tools/` | Scripts qui lancent les vérifications et **résument** leurs sorties (écrits en P1, T1.3) |
| `KICKOFF.md` | Le premier message à coller dans Claude Code |

## L'équipe et les modèles

Tu parles à l'**orchestrateur** (Opus, effort moyen). Il ne code presque pas : il découpe, délègue au modèle le moins cher capable de chaque tâche, intègre et vérifie.

| Modèle | Agents | Pour quoi |
|---|---|---|
| **Haiku** (le moins cher) | `Explore`, `verificateur`, `scribe` | Recherche dans le code, build et tests, suivi et commits |
| **Sonnet** | `dev-front`, `dev-supabase`, `testeur`, `relecteur` | L'essentiel du développement, les tests, les revues |
| **Opus** (le plus cher) | `architecte`, `securite` | Conception des phases, revue de la RLS et de tout ce qui touche à l'isolation entre clients |

**Mécanismes d'économie** :
- **Escalade automatique** : une tâche qui échoue 2 fois chez Sonnet passe sur Opus, puis on redescend.
- **Pas de Fable** sans ton accord explicite : Claude Code te le demandera.
- **Contexte léger par défaut** : chaque agent démarre avec un contexte minimal, lit les docs par section et renvoie un résumé de 15 lignes.
- **Build et tests résumés** : jamais de logs complets dans le contexte.

**Différences avec Brume** : un agent `securite` remplace le `perf-specialiste` ; il y a un dépôt GitHub (Pages et Actions l'exigent), mais `git push`, `git remote` et toute action sur la prod Supabase te sont demandés à chaque fois.

## Installation (une seule fois)

> **Fait le 2026-10-20 par Claude** : git initialisé (identité « Haris ») ; Node 24, Deno 2.9 et GitHub CLI installés, gh connecté à `Haris692` ; repo **public** https://github.com/Haris692/argos-ticketing créé et poussé (Pages refusé en privé sur l'offre gratuite) ; Pages activé en mode GitHub Actions.

1. **Outils restants** :
   - **Docker Desktop** (requis par Supabase en local) : `winget install Docker.DockerDesktop`, puis le lancer une fois (peut demander un redémarrage pour WSL2).
   - **Supabase CLI** : pas d'installation npm globale ; Scoop ou dépendance de dev du projet (`npx supabase`). Choisi en T1.1.
2. **Supabase** : créer un **nouveau** projet, distinct de celui de la v1 (l'offre gratuite suffit pour développer ; voir la question 6 de `docs/STATUS.md` pour la prod). **Ne donne jamais la clé secrète** à Claude.
3. **Trancher les questions ouvertes** de `docs/STATUS.md` (ou laisser Claude te les poser au lancement de P1).
4. **Lancer Claude Code dans ce dossier.** La session démarre automatiquement en mode orchestrateur.
5. **Coller le contenu de `KICKOFF.md`.**

## Au quotidien

| Commande | Effet |
|---|---|
| `/phase P1` | Démarre une phase. Les tâches s'enchaînent seules ; Claude te sollicite pour les décisions, puis s'arrête pour la recette |
| `/tache T1.3` | Fait une tâche précise |
| `/verif` | Build, lint et tests, avec un résumé (`rapide`, `complet`, `front`, `db`, `fn`) |
| `/recette retour` | À taper avec ton retour de recette, écrit librement (« je valide P1 » pour passer à la suite) |
| `/cloture`, puis `/clear` | En fin de session. La mémoire est dans `docs/STATUS.md`, rien n'est perdu |
| `/usage` | Ta consommation |
| `/context` | Ce qui remplit le contexte |
| `/model sonnet` | Passer l'orchestrateur sur Sonnet si tu approches de tes limites |

`Échap` interrompt Claude ; `/rewind` revient en arrière.

## Ce que Claude Code te demandera

- **Une recette à chaque fin de phase** : tester avec les comptes du seed, en clair, en sombre et sur mobile. Rien ne démarre avant ta validation.
- **Tes décisions** sur les points ambigus de la spec, les packages hors stack, et la configuration externe (Supabase, Postmark, DNS, GitHub).
- **Ton accord** pour chaque `git push` et chaque action sur la prod (`supabase link`, `db push`, `functions deploy`, `secrets set`).

## Personnaliser

| Pour changer… | Où |
|---|---|
| Le modèle d'un agent | La ligne `model:` de `.claude/agents/<agent>.md` |
| Une règle métier | `docs/ARGOS-TICKETING-SPEC.md` (et une ligne dans `docs/DECISIONS.md`) |
| Un paramètre d'arrondi du temps | `src/lib/time/config.ts` (après la P2) |
