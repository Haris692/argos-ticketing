---
paths:
  - "supabase/functions/**"
---

# Edge functions (Deno)

- **Authentifier d'abord** : `inbound-email` et `send-email` vérifient leur secret (comparaison à temps constant) ; `manage-contact` vérifie le JWT **et** le rôle admin. Sinon : 401, sans lire ni journaliser le corps.
- **Code partagé** dans `_shared/` : interface `EmailProvider` (Postmark en implémentation par défaut, simulable dans les tests), templates, helpers d'auth.
- **Clé secrète Supabase** : uniquement via l'environnement de la fonction ; jamais renvoyée, jamais journalisée.
- **Email entrant** (spec §8) : contenu non fiable, traité comme du texte brut ; le HTML entrant n'est jamais stocké ni rendu ; suivre l'ordre de traitement de la spec §8 à la lettre.
- **Templates** (spec §7, §11) : en français, HTML sobre + version texte brut systématique, objet `[REF] Titre`, jamais de pièce jointe sortante.
- **Logs** : pas de contenu d'email ni de données personnelles au-delà du nécessaire (identifiants, statut, motif).
- **Tests** : `deno test` pour chaque fonction de parsing, de nettoyage et de routage, sans appel réseau réel.
- **Déploiement** (`supabase functions deploy`) et secrets (`supabase secrets set`) : accord de Haris.
