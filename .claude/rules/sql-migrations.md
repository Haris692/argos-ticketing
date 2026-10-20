---
paths:
  - "supabase/migrations/**"
  - "supabase/seed.sql"
  - "supabase/config.toml"
---

# Migrations SQL et RLS

- **Jamais d'édition d'une migration existante** : toute modification = nouvelle migration (`supabase migration new <nom_en_snake_case>`).
- **Conventions** (spec §5) : PK `uuid default gen_random_uuid()`, `created_at timestamptz default now()`, `updated_at` par trigger, enums Postgres, FK `on delete restrict` sauf mention contraire.
- **RLS** : `alter table … enable row level security` dans la migration qui crée la table ; pas de policy = pas d'accès. Policies nommées, `USING` **et** `WITH CHECK` sur les écritures, helpers `public.is_admin()` / `public.current_org_id()`.
- **Colonnes protégées** : un trigger `BEFORE UPDATE` lève une exception si un non-admin modifie autre chose que les colonnes autorisées (liste blanche).
- **`SECURITY DEFINER`** : `set search_path = ''`, noms qualifiés (`public.tickets`), `revoke execute on function … from anon, public`. Obligatoire pour les triggers qui écrivent là où l'appelant n'a pas de droit (compteur d'organisation, `email_outbox`, réouverture).
- **Seed** : uniquement des données de test (comptes, organisations LSM et EUR, tickets). Aucun vrai email client, aucun secret.
- **Après chaque migration** : `supabase db reset`, régénération des types, puis le test pgTAP de la règle ajoutée (`docs/OUTILS.md`).
- **Prod** : `supabase db push` et toute commande `--linked` demandent l'accord de Haris.
