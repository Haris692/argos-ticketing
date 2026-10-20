# ARGOS TICKETING — Feuille de route

> Phases P1 à P8 (spec §12). Chaque tâche indique l'agent suggéré : l'orchestrateur applique la table de routage, et l'architecte peut découper une tâche (T2.2a, T2.2b…) dans `docs/phases/Px.md`.
> On coche `[x]` quand la tâche respecte la Definition of Done (CLAUDE.md). Le `scribe` tient cette page à jour.
> Lecture : ne lire que la section de la phase en cours (`grep -n "^## " docs/ROADMAP.md`).

## Règles

1. Chaque phase démarre par `/phase Px` : l'architecte rédige `docs/phases/Px.md` (schéma, interfaces, fichiers, ordre, risques), l'orchestrateur présente le plan à Haris et lui pose **en une seule fois** les questions qui bloquent cette phase.
2. Une phase se termine par : la suite complète (`verificateur`), une revue (`relecteur`, plus `securite` pour P1, P3, P4, P5, P6 et P8), la mise à jour des docs (`scribe`), une checklist de recette pour Haris (`/recette`), puis **arrêt**.
3. On n'entame pas la phase suivante avant la validation explicite de Haris (spec §2). Le tag `px` est posé à la validation.
4. Hors périmètre (spec §15), on ne code rien : on note l'idée dans DECISIONS, section « Idées ».

---

## P1 — Fondations

**Objectif** : un projet configuré, une boucle de vérification fiable, un schéma complet protégé par la RLS, la connexion par rôle.
**À lire** : spec §3 à §6, §11 (tokens), §12 (P1 et tests pgTAP), §13 ; `docs/OUTILS.md`.
**Prérequis (Haris)** : `README_HARIS.md`, étapes 1 à 4 ; questions ouvertes de STATUS tranchées.

- [ ] **T1.1 Inventaire de l'environnement** — `verificateur`. Relever les versions (Node, npm, Docker, Supabase CLI, Deno, git, gh) et remplir OUTILS §1. *Critère* : les manques sont listés pour Haris en une fois.
- [ ] **T1.2 Scaffold front** — `dev-front`. Vite + React 19 + TS strict, Tailwind 4, shadcn/ui, ESLint, Prettier, Vitest, arborescence de la spec §3, HashRouter, `.env.example`. *Critère* : `npm run build`, `lint` et `test` passent sur un projet vide.
- [ ] **T1.3 Boucle de vérification** — `testeur`, puis validation par `verificateur`. Script `tools/verif.ps1` (portées rapide / complet / front / db / fn) qui écrit les sorties dans `.ci/` et n'affiche qu'un résumé ; commandes exactes dans OUTILS §2 et §3. *Critère* : `/verif` répond en 15 lignes maximum.
- [ ] **T1.4 Supabase local et schéma** — `dev-supabase`. `supabase init`, enums, tables (spec §5), `is_admin()`, `current_org_id()`, `get_ticket_time_entries()`, triggers `updated_at`, numérotation, dates de statut. *Critère* : `supabase db reset` OK, types générés.
- [ ] **T1.5 RLS, triggers de garde et Storage** — `dev-supabase`, revue `securite`. Policies de la spec §6, garde de colonnes en liste blanche, valeurs forcées à l'insertion, réouverture sur commentaire client, bucket `ticket-attachments` et ses policies. *Critère* : revue `securite` sans BLOQUANT.
- [ ] **T1.6 Tests pgTAP** — `testeur`. La liste minimale de la spec §12, plus un test par policy. *Critère* : `supabase test db` vert.
- [ ] **T1.7 Seed** — `dev-supabase`. Admin, LSM et EUR, 2 contacts par organisation, ~10 tickets variés avec commentaires et entrées de temps. *Critère* : `supabase db reset` charge le seed ; comptes documentés dans OUTILS §4.
- [ ] **T1.8 Design system et layout** — `dev-front`. Tokens clair / sombre (spec §11), bascule clair / sombre / système, barre de navigation en verre dépoli, layout admin et client. *Critère* : rendu vérifié en clair, en sombre et à 375 px.
- [ ] **T1.9 Authentification** — `dev-front`. Connexion, mot de passe oublié, définition du mot de passe (invitation et reset) compatibles HashRouter, redirection selon le rôle, garde de routes. *Critère* : l'admin et un contact de chaque client se connectent et arrivent sur le bon écran.
- [ ] **T1.10 Déploiement GitHub Pages** — `dev-front`. Workflow `.github/workflows/deploy.yml` (build avec les variables `VITE_*` depuis les secrets GitHub). *Critère* : workflow valide ; activation et premier push faits par Haris.

**Sortie P1** : connexion fonctionnelle pour l'admin et un contact de chaque client, tests RLS verts. Recette de Haris.

## P2 — Cœur admin

**Objectif** : Haris traite les tickets et saisit son temps.
**À lire** : spec §9 (admin 1 et 2), §10, §11.

- [ ] **T2.1 Moteur de temps** — `dev-front` (logique pure), puis `testeur`. `src/lib/time/` : `config.ts`, `computeTicketTime`, `formatDuration`, avec tous les tests de la spec §10 qui les concernent.
- [ ] **T2.2 Liste des tickets admin** — `dev-front`. Filtres, recherche, tri, compteurs, badges (spec §9 admin 1).
- [ ] **T2.3 Panneau de détail** — `dev-front`. Panneau latéral, URL `#/tickets/REF`, statut / priorité / catégorie en un clic, fil chronologique.
- [ ] **T2.4 Commentaires** — `dev-front`. Ajout et affichage, rendu en texte brut.
- [ ] **T2.5 Bloc temps admin** — `dev-front`. Liste, formulaire (boutons 15 / 30 / 45 / 60 / 90, date par défaut Europe/Paris), totaux réel / comptabilisé / facturable via le moteur.

**Sortie P2** : recette de Haris sur le traitement d'un ticket de bout en bout.

## P3 — Portail client et gestion des contacts

**Objectif** : les contacts clients ouvrent et suivent leurs tickets ; Haris invite et désactive des contacts.
**À lire** : spec §4, §6, §9 (client, admin 3).

- [ ] **T3.1 Mes tickets et nouveau ticket** — `dev-front`.
- [ ] **T3.2 Détail client et temps passé** — `dev-front`. Lecture via `get_ticket_time_entries()`, ticket fermé en lecture seule.
- [ ] **T3.3 Réouverture automatique** — `dev-supabase` + `testeur` (si pas déjà couverte en P1).
- [ ] **T3.4 Edge function `manage-contact`** — `dev-supabase`, revue `securite`. Invitation, désactivation (ban Auth), réactivation ; JWT + rôle admin vérifiés.
- [ ] **T3.5 Écran Organisations & contacts** — `dev-front`.
- [ ] **T3.6 Profil client** — `dev-front`. Nom, notifications, mot de passe.

**Sortie P3** : recette avec un vrai parcours client (sur des comptes de test).

## P4 — Pièces jointes

**À lire** : spec §6 (Storage), §9.

- [ ] **T4.1 Upload web** — `dev-front`. Bouton, glisser-déposer, coller une capture, validations type et taille, noms nettoyés.
- [ ] **T4.2 Affichage et téléchargement** — `dev-front`. Miniatures, URL signée de 5 minutes.
- [ ] **T4.3 Tests Storage** — `testeur`. Isolation entre organisations, ticket fermé, types refusés ; revue `securite`.

## P5 — Notifications sortantes

**Prérequis (Haris)** : domaine, compte Postmark, DNS (spec §16).
**À lire** : spec §7, §11 (emails).

- [ ] **T5.1 `email_outbox` et triggers** — `dev-supabase`.
- [ ] **T5.2 `EmailProvider`, Postmark et templates** — `dev-supabase`.
- [ ] **T5.3 Edge function `send-email` et Database Webhook** — `dev-supabase`, revue `securite`.
- [ ] **T5.4 Relance pg_cron et préférences de notification** — `dev-supabase` + `dev-front`.
- [ ] **T5.5 Tests** — `testeur` (pgTAP sur les triggers, Deno sur les templates et l'envoi simulé).

## P6 — Email entrant

**À lire** : spec §8, §9 (admin 4).

- [ ] **T6.1 `inbound-email` : auth, dédup, filtres, anti-usurpation, limite** — `dev-supabase`.
- [ ] **T6.2 Identification, routage, nettoyage du corps, pièces jointes** — `dev-supabase`.
- [ ] **T6.3 Tests Deno** — `testeur`. Parsing, nettoyage, routage, quarantaine.
- [ ] **T6.4 Écran Quarantaine** — `dev-front`.
- [ ] **T6.5 Revue** — `securite` sur l'ensemble du flux entrant.

## P7 — Export du temps

**À lire** : spec §9 (admin 5), §10.

- [ ] **T7.1 `computeMonthlyTimesheet` et `timesheetToCsv`** — `dev-front`, puis `testeur` (tous les tests de la spec §10).
- [ ] **T7.2 Écran Export** — `dev-front`.

## P8 — Finitions et mise en production

**À lire** : spec §12 (P8), §13, §16.

- [ ] **T8.1 Recherche et filtres avancés, polish responsive et dark mode, états vides** — `dev-front`.
- [ ] **T8.2 MFA TOTP admin** — `dev-front`.
- [ ] **T8.3 Purge pg_cron des `raw_payload` à 30 jours** — `dev-supabase`.
- [ ] **T8.4 CSP en balise `<meta>`** — `dev-front`, revue `securite`.
- [ ] **T8.5 Checklist de mise en prod** — orchestrateur avec Haris (spec §12, P8) : actions sur la prod faites ou validées par Haris, `supabase test db` rejoué sur la prod, test de bout en bout avant d'inviter un vrai client.
