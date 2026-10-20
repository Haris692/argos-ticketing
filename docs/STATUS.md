# STATUS — état courant du projet

> Mémoire de travail entre les sessions. Tenue à jour par l'agent `scribe` à la fin de chaque tâche.
> **60 lignes maximum** : on résume, on ne raconte pas. L'historique, c'est git.

## Où on en est
- **Phase** : aucune commencée. Kit Claude Code en place (organisation reprise de Brume, D-010).
- **Repo** : git local initialisé (branche `main`), pas encore de dépôt GitHub.
- **Environnement** : Node, npm, Docker, Supabase CLI, Deno et gh **absents** de la machine (constat du 2026-10-20, à confirmer en T1.1).

## Reprise à froid
- **Prochaine étape** : Haris suit `README_HARIS.md` (étapes 1 à 4), puis colle `KICKOFF.md` dans une nouvelle session.
- **À faire par Haris** : installer les outils, créer le projet Supabase et le repo GitHub (spec §16), trancher les questions ci-dessous.

## Questions ouvertes pour Haris (à trancher avant ou pendant /phase P1)
1. **Contact désactivé** : un contact désactivé dont le domaine est autorisé peut encore créer des tickets par email (§8.5). Reco : quarantaine.
2. **Demandeur sans compte** : il reçoit un lien vers le portail qu'il ne peut pas ouvrir, et ne voit pas les pièces jointes. On l'accepte, ou on adapte ses emails ?
3. **Nom de Haris côté client** : la RLS sur `profiles` empêche un client de voir le profil admin (org nulle) → commentaires de Haris sans auteur. Reco : vue ou fonction exposant seulement le nom de l'admin.
4. **Arrondi facturable / non facturable** : 10 min facturables + 10 min non facturables le même jour donnent 30 min comptabilisées (deux groupes, minimum 15 chacun). Voulu ?
5. **`reply_token` lisible** par les contacts de l'org via l'API (la RLS filtre les lignes, pas les colonnes). Risque faible ; reco : privilèges par colonne ou vue.
6. **Supabase gratuit** : le projet se met en pause après 7 jours sans activité (emails entrants perdus). Offre Pro avant la mise en prod ?

## Points techniques à intégrer au plan P1 (architecte)
- Triggers qui écrivent là où le client n'a pas de droit (compteur d'org, `email_outbox`, réouverture) : `SECURITY DEFINER`.
- La réouverture sur commentaire client modifie `status` : prévoir l'exception dans le trigger de garde de colonnes.
- Garde de colonnes en **liste blanche** (client : `title`, `description`) ; à l'insertion, forcer aussi `reply_token`, `number`, `reference`.
- HashRouter et liens Supabase (invitation, reset) : flux PKCE ou modèles d'email avec `token_hash`, pas de jeton dans le fragment.
- Relance des emails en échec (P5) : le Database Webhook ne réagit qu'à l'INSERT → pg_cron + `pg_net`.
- À vérifier en P6 : quels résultats SPF / DKIM / DMARC Postmark transmet réellement.

## Ce qui marche
- Rien de code pour l'instant.

## Points à surveiller
- CSP en `<meta>` : `frame-ancestors` ignoré, pas de protection contre l'intégration dans une iframe sur GitHub Pages.

## Dette technique
- Aucune.
