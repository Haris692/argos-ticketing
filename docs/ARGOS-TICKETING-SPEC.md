# Argos Ticketing — Spécification v2 (handoff Claude Code)

> **Version 2 — octobre 2026.** Remplace la spec d'août.
> Nouveautés v2 : création de tickets par email, notifications email, pièces jointes, temps passé visible côté client, export mensuel du temps pour la facturation.

---

## 1. Contexte

Haris est consultant IT / cybersécurité indépendant (Lyon). Il gère plusieurs clients freelance en parallèle. Au lancement, deux clients : **LSM Logistics** et **Eurofret**. D'autres viendront.

Il a besoin d'un outil de ticketing où :

- **Haris (admin unique)** centralise toutes les demandes de tous ses clients ;
- **les contacts de chaque client** ouvrent et suivent leurs tickets via un **portail web** ou **par email** ;
- **chaque ticket porte le temps passé**, visible par le client, et exportable par mois pour la facturation.

Argos Ticketing est le premier module de la suite **Argos**. Un module Facturation viendra plus tard : il réutilisera tel quel le moteur de calcul du temps défini ici (section 10).

---

## 2. Comment utiliser ce document

- L'implémentation se fait **phase par phase** (section 12). Une phase = un lot livrable et testable.
- **À la fin de chaque phase : s'arrêter**, résumer ce qui a été fait, lister ce que Haris doit tester ou configurer, et attendre sa validation avant la phase suivante.
- Si un point est ambigu ou semble contradictoire : **poser la question** plutôt qu'inventer.
- Les choix déjà tranchés par défaut sont listés en section 14. Ils peuvent être remis en cause par Haris, pas par l'implémentation.

---

## 3. Stack technique

| Couche | Choix |
|---|---|
| Frontend | React 19 + Vite + TypeScript (strict) + Tailwind CSS 4 + shadcn/ui |
| Routing | React Router en **HashRouter** (GitHub Pages ne gère pas les rewrites) |
| Data / cache | `@supabase/supabase-js` + TanStack Query |
| Formulaires | react-hook-form + zod |
| Icônes | lucide-react |
| Backend | Supabase : Auth, Postgres, RLS, Storage, Edge Functions (Deno), Database Webhooks, pg_cron |
| Email | Fournisseur transactionnel gérant **envoi + réception (inbound webhook)**. Par défaut **Postmark**, derrière une interface `EmailProvider` pour pouvoir en changer sans toucher au reste |
| Hébergement front | GitHub Pages, déployé par GitHub Actions |
| Dev local | Supabase CLI (`supabase start`), migrations versionnées dans `supabase/migrations/` |
| Tests | **Vitest** (logique métier, `src/lib`), **pgTAP** via `supabase test db` (RLS, triggers, fonctions), **Deno test** (edge functions : parsing et routage email) |

### Structure du repo

```
argos-ticketing/
├── CLAUDE.md
├── docs/ARGOS-TICKETING-SPEC.md
├── src/
│   ├── app/                 # routes, layouts, garde par rôle
│   ├── features/
│   │   ├── auth/
│   │   ├── tickets/
│   │   ├── comments/
│   │   ├── attachments/
│   │   ├── time/
│   │   ├── organizations/   # orgs + contacts + invitations
│   │   ├── quarantine/
│   │   └── export/
│   ├── components/ui/       # shadcn/ui
│   ├── lib/
│   │   ├── supabase.ts
│   │   ├── database.types.ts   # généré par la CLI
│   │   └── time/               # moteur de calcul pur + tests
│   └── styles/              # tokens de design (CSS vars light/dark)
├── supabase/
│   ├── migrations/
│   ├── functions/
│   │   ├── _shared/         # EmailProvider, templates, helpers d'auth
│   │   ├── send-email/
│   │   ├── inbound-email/
│   │   └── manage-contact/
│   ├── tests/               # pgTAP
│   └── seed.sql
└── .github/workflows/deploy.yml
```

### Variables d'environnement

**Front (`.env`, publiques par nature — préfixe `VITE_`)**
- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY` (clé publique anon / publishable)
- `VITE_SUPPORT_ADDRESS` (adresse affichée aux clients, ex. `support@<domaine>`)

**Edge functions (`supabase secrets set`, jamais dans le repo)**
- `EMAIL_PROVIDER_API_KEY`
- `INBOUND_WEBHOOK_SECRET` (authentifie le webhook entrant)
- `OUTBOX_WEBHOOK_SECRET` (authentifie le Database Webhook vers `send-email`)
- `APP_URL` (URL GitHub Pages du portail)
- `SUPPORT_ADDRESS`, `INBOUND_DOMAIN`, `ADMIN_EMAIL`
- La clé secrète Supabase (service_role / secret) est fournie automatiquement aux edge functions : **elle n'existe nulle part ailleurs.**

---

## 4. Rôles et accès

| Rôle | Qui | Droits |
|---|---|---|
| `admin` | Haris uniquement | Tout voir, tout modifier, gérer organisations et contacts, saisir le temps, exporter |
| `client` | Contact rattaché à une organisation | Voir **tous les tickets de son organisation**, en créer, commenter, joindre des fichiers, voir le temps comptabilisé et les notes publiques |

- **Pas d'inscription publique** : signups désactivés dans Supabase Auth. Les comptes clients sont créés **par invitation** depuis l'écran admin (edge function `manage-contact`).
- Auth : email + mot de passe, réinitialisation par email, page de définition du mot de passe après invitation.
- **MFA TOTP obligatoire pour le compte admin** (phase 8).
- Un contact peut être **désactivé** (`profiles.is_active = false`) : il perd immédiatement tout accès (géré au niveau RLS) et son compte Auth est banni via l'API admin.
- Les expéditeurs email sans compte sont traités en section 8.

---

## 5. Modèle de données

**Conventions** : PK `uuid default gen_random_uuid()`, `created_at timestamptz default now()`, `updated_at` maintenu par trigger, enums Postgres, FK `on delete restrict` sauf mention contraire.

### `organizations`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK |
| name | text | « LSM Logistics », « Eurofret » |
| code | text unique | 3–5 majuscules : `LSM`, `EUR`. Préfixe des références tickets |
| ticket_counter | int default 0 | Compteur de numérotation (trigger) |
| allowed_email_domains | text[] default '{}' | Domaines autorisés à ouvrir un ticket par email sans compte |
| created_at | timestamptz | |

### `profiles`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK = `auth.users.id` |
| organization_id | uuid null | FK `organizations`. Null pour l'admin |
| role | enum `user_role` (`admin`, `client`) | |
| full_name | text | |
| email | text unique | |
| notify_email | boolean default true | |
| is_active | boolean default true | |
| created_at | timestamptz | |

Contrainte : `role = 'client'` ⇒ `organization_id is not null`.

### `tickets`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK |
| organization_id | uuid | FK |
| number | int | Séquence **par organisation**, posée par trigger |
| reference | text unique | `LSM-0042`, posée par trigger à l'insertion |
| created_by | uuid null | FK `profiles`. Null si ouvert par email par un expéditeur sans compte |
| requester_email | text null | Email du demandeur sans compte |
| title | text | |
| description | text | Texte brut |
| status | enum `ticket_status` : `nouveau`, `en_cours`, `en_attente`, `resolu`, `ferme` | |
| priority | enum `ticket_priority` : `basse`, `normale`, `haute`, `urgente` | Le client suggère, l'admin corrige |
| category | text | Valeurs proposées : M365, Réseau, Sécurité, Poste de travail, Autre |
| source | enum `ticket_source` : `web`, `email` | |
| reply_token | uuid unique default gen_random_uuid() | Jeton non devinable pour l'adresse de réponse email. Jamais affiché dans l'UI |
| resolved_at | timestamptz null | |
| closed_at | timestamptz null | |
| created_at / updated_at | timestamptz | |

### `ticket_comments`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK |
| ticket_id | uuid | FK `on delete cascade` |
| author_id | uuid null | FK `profiles`. Null si email d'un expéditeur sans compte |
| author_email | text null | |
| body | text | Texte brut, toujours rendu échappé |
| source | enum `ticket_source` | |
| created_at | timestamptz | |

### `ticket_attachments`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK |
| ticket_id | uuid | FK `on delete cascade` |
| comment_id | uuid null | FK si joint à un commentaire |
| uploaded_by | uuid null | FK `profiles` |
| uploaded_by_email | text null | Cas email sans compte |
| storage_path | text unique | `{organization_id}/{ticket_id}/{uuid}-{nom_nettoyé}` |
| file_name | text | Nom d'origine (affichage) |
| mime_type | text | |
| size_bytes | int | |
| created_at | timestamptz | |

### `time_entries`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK |
| ticket_id | uuid | FK |
| logged_by | uuid | FK `profiles` (toujours l'admin) |
| worked_on | date | Jour de l'intervention. Par défaut : aujourd'hui en **Europe/Paris** |
| duration_minutes | int, check > 0 | **Durée réelle, jamais arrondie au stockage** |
| is_billable | boolean default true | |
| public_note | text null | Visible par le client |
| internal_note | text null | **Admin uniquement, jamais exposée au client** |
| created_at / updated_at | timestamptz | |

Aucun total stocké : tout est calculé à la volée par le moteur de la section 10.

### `email_outbox`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK |
| to_email | text | |
| template | text | `ticket_created_admin`, `ticket_created_ack`, `comment_added`, `status_changed` |
| ticket_id | uuid null | |
| payload | jsonb | Données du template |
| status | text : `pending`, `sent`, `failed` | |
| attempts | int default 0 | |
| last_error | text null | |
| provider_message_id | text null | |
| created_at / sent_at | timestamptz | |

### `inbound_emails`
| Colonne | Type | Notes |
|---|---|---|
| id | uuid | PK |
| message_id | text unique | Header `Message-ID`, sert à dédupliquer |
| received_at | timestamptz | |
| from_email | text | |
| subject | text | |
| status | text : `accepted`, `quarantined`, `rejected`, `ignored` | |
| reason | text null | Motif lisible (« expéditeur inconnu », « DMARC en échec »…) |
| auth_result | text null | Résumé SPF / DKIM / DMARC |
| ticket_id / comment_id | uuid null | Résultat du traitement |
| raw_payload | jsonb null | **Purgé après 30 jours** (pg_cron) |

### Fonctions SQL
- `is_admin()` et `current_org_id()` : `SECURITY DEFINER`, `STABLE`, `set search_path = ''`. Utilisées dans **toutes** les policies (évite la récursion RLS sur `profiles`). `current_org_id()` renvoie null si le profil est inactif.
- `get_ticket_time_entries(p_ticket_id uuid)` : `SECURITY DEFINER`. Vérifie que l'appelant est admin ou que le ticket appartient à `current_org_id()`. Renvoie uniquement `id, worked_on, duration_minutes, is_billable, public_note`. **Jamais `internal_note`.**
- `execute` révoqué pour `anon` sur toutes ces fonctions.

---

## 6. Row Level Security et règles d'intégrité

**RLS activée sur toutes les tables du schéma `public`. Pas de policy = pas d'accès.**

| Table | Client | Admin |
|---|---|---|
| `organizations` | SELECT de sa propre org | Tout |
| `profiles` | SELECT de soi-même et des contacts de son org (affichage des noms). UPDATE de soi-même, limité à `full_name` et `notify_email` | Tout |
| `tickets` | SELECT des tickets de son org. INSERT dans son org avec `created_by = auth.uid()`. UPDATE de **ses propres tickets**, uniquement `title` et `description`, uniquement tant que `status = 'nouveau'`. Pas de DELETE | Tout |
| `ticket_comments` | SELECT si ticket visible. INSERT sur un ticket de son org, `author_id = auth.uid()`, **interdit si ticket fermé**. Pas d'UPDATE / DELETE | Tout |
| `ticket_attachments` | SELECT si ticket visible. INSERT sur un ticket de son org (non fermé). Pas de DELETE | Tout |
| `time_entries` | **Aucun accès direct** : lecture via `get_ticket_time_entries()` | Tout |
| `email_outbox`, `inbound_emails` | Aucun accès | SELECT. Écriture par triggers et edge functions |

RLS ne filtre que des lignes : les restrictions par colonne sont imposées par **triggers** qui lèvent une exception si un non-admin tente de modifier :
- `tickets` : `status`, `priority`, `category`, `organization_id`, `created_by`, `reference`, `number`, `reply_token` ;
- `profiles` : `role`, `organization_id`, `email`, `is_active`.

À l'INSERT client, un trigger force `status = 'nouveau'` et `source = 'web'`.

### Storage
- Bucket **privé** `ticket-attachments`.
- Policies sur `storage.objects` : SELECT / INSERT si admin, ou si le premier segment du chemin = `current_org_id()` **et** le ticket (2ᵉ segment) appartient à cette org. DELETE : admin uniquement.
- Limites du bucket : **10 Mo par fichier**, types autorisés : PNG, JPEG, WEBP, GIF, HEIC, PDF, TXT, LOG, CSV, DOCX, XLSX. Archives et exécutables refusés.
- Téléchargement uniquement par **URL signée de 5 minutes**.
- Noms de fichiers nettoyés (pas de `../`, caractères spéciaux remplacés).

### Automatisations (triggers)
- Numérotation : à l'insertion d'un ticket, incrément de `organizations.ticket_counter` (verrou de ligne), puis `number` et `reference = code || '-' || lpad(number, 4, '0')`.
- `updated_at` sur les tables concernées.
- Commentaire **client** sur un ticket `en_attente` ou `resolu` → repasse `en_cours` (et `resolved_at = null`).
- Passage à `resolu` → `resolved_at = now()` ; passage à `ferme` → `closed_at = now()`.
- Insertion des lignes `email_outbox` (section 7).

---

## 7. Notifications email sortantes

**Architecture** : les triggers Postgres insèrent dans `email_outbox` → Database Webhook Supabase sur INSERT → edge function `send-email` (authentifiée par `OUTBOX_WEBHOOK_SECRET`) → fournisseur. En cas d'échec : `status = 'failed'`, `attempts + 1`, `last_error`. Une tâche **pg_cron toutes les 10 min** relance les lignes `failed` avec `attempts < 3`.

| Événement | Destinataires |
|---|---|
| Nouveau ticket (web ou email) | Haris + **accusé de réception** au demandeur |
| Commentaire d'un client | Haris |
| Commentaire de Haris | Créateur du ticket (ou `requester_email`) |
| Statut passé à `en_attente`, `resolu` ou `ferme` | Créateur du ticket (ou `requester_email`) |

**Règles**
- Ne jamais notifier l'auteur de l'action.
- Respecter `notify_email` (les demandeurs sans compte reçoivent toujours les emails de leur ticket).
- Jamais d'email vers un expéditeur en quarantaine.

**Contenu**
- Objet : `[LSM-0042] Titre du ticket`.
- Corps HTML sobre (même direction artistique que l'app, section 11) **+ version texte brut systématique**.
- Extrait du message, lien direct vers le ticket dans le portail (`{APP_URL}/#/tickets/LSM-0042`), mention « Répondez directement à cet email pour ajouter un commentaire ».
- `Reply-To: support+{reply_token}@{INBOUND_DOMAIN}` (le fournisseur doit gérer le plus-addressing).
- Pas de pièces jointes dans les emails sortants : lien vers le portail.
- Templates en TypeScript dans `supabase/functions/_shared/templates/`, en français.

---

## 8. Création de tickets par email (inbound)

**Flux** : le client écrit à `support@{INBOUND_DOMAIN}` → le fournisseur parse l'email → POST vers l'edge function `inbound-email`. Cette fonction est déployée **sans vérification JWT** et s'authentifie par `INBOUND_WEBHOOK_SECRET` (basic auth dans l'URL du webhook ou header secret). Sinon : 401.

### Traitement, dans l'ordre
1. **Authentifier** le webhook, puis **dédupliquer** par `Message-ID` (déjà vu → 200 sans rien faire). Journaliser dans `inbound_emails`.
2. **Ignorer** (status `ignored`) : auto-réponses (`Auto-Submitted` ≠ `no`, `X-Autoreply`, `Precedence: bulk / auto_reply / list`), bounces, emails émis par nos propres adresses (anti-boucle).
3. **Anti-usurpation** : lire les résultats SPF / DKIM / DMARC fournis par le fournisseur. DMARC en échec, ou SPF et DKIM tous deux en échec → **quarantaine**.
4. **Limiter** : plus de 20 emails / heure d'un même expéditeur → quarantaine.
5. **Identifier l'expéditeur** :
   - email d'un profil client actif → son organisation ;
   - sinon, domaine présent dans `allowed_email_domains` d'une org → accepté, `created_by = null`, `requester_email` renseigné, ticket signalé « contact non enregistré » pour que Haris puisse l'inviter ;
   - email de Haris lui-même → quarantaine (il choisit l'organisation à la main) ;
   - sinon → **quarantaine**.
6. **Router** :
   - adresse `support+{reply_token}@…` correspondant à un ticket **de l'org de l'expéditeur** → commentaire sur ce ticket (si ticket fermé : nouveau ticket mentionnant l'ancienne référence) ;
   - sinon, objet contenant `[XXX-0000]` d'un ticket de la même org → commentaire ;
   - sinon → **nouveau ticket** : titre = objet (ou « (sans objet) »), description = corps nettoyé, priorité `normale`, catégorie « Autre », source `email`.
   - Un `reply_token` qui pointe vers une autre org que celle de l'expéditeur → quarantaine.
7. **Nettoyer le corps** : utiliser la réponse « nettoyée » fournie par le fournisseur si elle existe ; sinon couper au premier marqueur de citation (« Le … a écrit : », « On … wrote: », lignes `>`). Préférer `text/plain` ; si seulement du HTML, le convertir en texte. **Le HTML entrant n'est jamais stocké ni rendu.**
8. **Pièces jointes** : mêmes règles que le web (types, 10 Mo). Images inline de signature (Content-ID et < 20 Ko) ignorées. Fichiers refusés listés dans le commentaire : « Pièce jointe ignorée : facture.exe (type non autorisé) ».
9. **Notifier** normalement (accusé de réception, alerte Haris). Pour une quarantaine : alerte Haris uniquement, **jamais de réponse à l'expéditeur**.

Le contenu des emails entrants est **non fiable** : traité comme du texte brut partout.

---

## 9. Écrans

### Communs
- Connexion, mot de passe oublié, définition du mot de passe (après invitation ou reset).
- Barre de navigation : logo Argos, navigation, bascule clair / sombre / système, menu compte.

### Admin
1. **Tickets** (accueil) — liste tous clients confondus. Filtres : organisation, statut, priorité, catégorie, source. Recherche par référence ou titre. Tri par création / dernière mise à jour. Compteurs : nouveaux, en cours, en attente, en quarantaine. Badge « contact non enregistré ».
2. **Détail ticket** — panneau latéral glissant depuis la droite, URL dédiée `#/tickets/LSM-0042`.
   - En-tête : référence, titre, organisation, demandeur, source.
   - Statut, priorité, catégorie modifiables en un clic.
   - Fil chronologique : description, commentaires, pièces jointes (miniatures pour les images).
   - Ajout de commentaire + pièces jointes.
   - **Bloc temps** : liste des entrées (date, durée réelle, facturable, notes) ; formulaire : date (défaut aujourd'hui), durée avec boutons rapides 15 / 30 / 45 / 60 / 90 min + saisie libre, facturable (oui par défaut), note publique, note interne. Totaux : **réel**, **comptabilisé**, **facturable**.
3. **Organisations & contacts** — CRUD des organisations (nom, code, domaines autorisés). Liste des contacts, invitation (nom + email), désactivation / réactivation.
4. **Quarantaine** — liste des emails en quarantaine avec motif. Aperçu en texte brut. Actions : attribuer à une organisation (crée le ticket), rejeter, ajouter le domaine aux domaines autorisés.
5. **Export du temps** — choix organisation + mois → aperçu par ticket et par jour, totaux réel / comptabilisé / facturable, bouton **Exporter en CSV**.
6. **Paramètres** — notifications, mot de passe, MFA.

### Client
1. **Mes tickets** — tickets de l'organisation, filtre statut, recherche, bouton « Nouveau ticket ». Rappel discret : « Vous pouvez aussi écrire à support@… ».
2. **Détail ticket** — description, fil, ajout de commentaire + pièces jointes. Si fermé : saisie désactivée, message « Ce ticket est fermé. Ouvrez un nouveau ticket si besoin. »
   **Bloc « Temps passé »** : entrées (date, durée comptabilisée, note publique, badge « non facturé » le cas échéant) et total comptabilisé.
3. **Nouveau ticket** — titre, description, catégorie, priorité suggérée, pièces jointes (bouton, glisser-déposer, **coller une capture d'écran avec Ctrl/Cmd + V**).
4. **Profil** — nom, notifications email, mot de passe.

---

## 10. Moteur de calcul du temps (module pur)

Emplacement : `src/lib/time/`. **Aucune dépendance à React ni à Supabase** : ce module sera réutilisé tel quel par le futur module Facturation.

**Le même module sert à l'affichage client, à l'affichage admin et à l'export** : le client voit exactement ce qui sera facturé.

### Règles
- Stockage brut en minutes.
- **Temps comptabilisé** : on regroupe les entrées par **(ticket, jour, facturable)**, on **somme**, puis on arrondit la somme. On n'arrondit jamais les entrées une par une.
- Arrondi au palier `ROUNDING_STEP_MINUTES` (15) selon `ROUNDING_MODE` (`'nearest'` par défaut, `'up'` disponible).
- Minimum `MIN_COUNTED_MINUTES` (15) dès qu'un groupe a une somme > 0.
- Les entrées non facturables comptent dans le réel et le comptabilisé, mais pas dans le facturable.
- Tous les paramètres dans `src/lib/time/config.ts`.

### Fonctions
- `computeTicketTime(entries) → { realMinutes, countedMinutes, billableMinutes }`
- `computeMonthlyTimesheet(entries, tickets, { organizationId, month }) → { lignes par jour et par ticket, totaux }`
- `timesheetToCsv(timesheet) → string` : UTF-8 **avec BOM**, séparateur `;`, décimales à virgule (compatibilité Excel FR), durées en heures décimales **et** en `hh:mm`. Colonnes : date, référence, titre, durée réelle, durée comptabilisée, facturable, note publique. Ligne de totaux en fin de fichier.
- `formatDuration(minutes) → "1 h 15"`

### Tests Vitest obligatoires
- Somme simple d'entrées.
- Deux entrées **même ticket, même jour** : 10 + 10 min → somme 20 → comptabilisé 15 en `nearest` (et non 15 + 15 = 30).
- Même ticket, jours différents : arrondis séparés.
- Modes `nearest` et `up`, valeurs pile sur un palier, minimum appliqué.
- Facturable / non facturable séparés.
- Mois : bornes (31 → 1ᵉʳ), entrées hors mois ignorées, filtre par organisation.
- CSV : BOM, séparateur, virgule décimale, échappement des titres contenant `;`, `"` ou retour à la ligne.
- `formatDuration` : 0, 15, 60, 75, 600 minutes.

---

## 11. Design system — direction « Apple »

L'objectif : un outil qui a l'air d'un produit Apple interne, pas d'un dashboard SaaS générique.

### Typographie
- `-apple-system, BlinkMacSystemFont, "SF Pro Text", Inter, system-ui, sans-serif` (Inter en fallback web).
- Corps 15–16 px, line-height 1.5. Titres semibold, léger letter-spacing négatif. Jamais de texte < 13 px.
- Chiffres tabulaires pour les références et les durées.

### Couleurs (CSS variables, light / dark)
| Token | Light | Dark |
|---|---|---|
| Fond | `#FAFAFA` | `#0A0A0A` |
| Surface | `#FFFFFF` | `#1C1C1E` |
| Surface élevée | `#FFFFFF` + ombre | `#2C2C2E` |
| Texte principal | `#1D1D1F` | `#F5F5F7` |
| Texte secondaire | `#6E6E73` | `#98989D` |
| Séparateur | `rgba(0,0,0,0.08)` | `rgba(255,255,255,0.10)` |
| Accent (unique) | `#0071E3` | `#0A84FF` |

- **Une seule couleur d'accent**, réservée aux actions primaires, au focus et à la sélection.
- **Statuts en badges discrets** (fond très légèrement teinté + texte) : `nouveau` = accent ; `en cours` = texte principal avec point accent ; `en attente` = gris ; `résolu` = gris + coche ; `fermé` = gris clair.
- Seule exception : priorité **urgente**, petit point rouge `#FF3B30`, jamais en aplat.
- Dark mode natif : vraies surfaces élevées, pas un filtre inversé. Thème clair / sombre / système dès la phase 1.

### Composants et profondeur
- Cards : coins arrondis 14 px, ombre très légère (`0 1px 2px rgba(0,0,0,.04), 0 4px 16px rgba(0,0,0,.04)`), au plus un séparateur 1 px.
- **Verre dépoli** (backdrop-blur ~20 px + fond semi-transparent) **uniquement sur les éléments flottants** : barre de navigation, panneau de détail, modales, menus. Jamais sur le contenu.
- Détail ticket en **panneau latéral** glissant depuis la droite (~560 px sur desktop, plein écran sur mobile), liste visible derrière.
- Listes aérées : lignes ~56 px, une info principale + une ligne secondaire grise.
- Boutons : primaire plein accent (rayon 10 px), secondaire gris translucide. Icônes Lucide trait fin.
- Champs : fond légèrement teinté, pas de bordure lourde, anneau de focus accent.
- États vides soignés : icône, phrase courte, action.

### Mouvement
- 150–250 ms, `cubic-bezier(0.25, 0.1, 0.25, 1)`. Panneau en slide + fondu.
- Respect de `prefers-reduced-motion`.

### À éviter
Densité façon Jira, plusieurs couleurs saturées, bordures épaisses, dégradés décoratifs, ombres dures, emojis dans l'interface.

### Mobile
Usage terrain (Haris est souvent en déplacement) : tout est utilisable à 375 px de large, actions principales atteignables au pouce.

### Emails
Même sobriété : fond blanc, une colonne de 560 px, typo système, un seul bouton accent « Voir le ticket », version texte brut systématique.

---

## 12. Phasage

> À la fin de chaque phase : build OK, tests verts, résumé, liste de ce que Haris doit tester ou configurer, **puis arrêt**.

**Phase 1 — Fondations**
- Scaffold Vite + React + TS + Tailwind + shadcn/ui, ESLint, Prettier, structure de dossiers.
- Tokens de design light / dark (section 11) et layout de base.
- Supabase CLI en local. Migrations : tous les enums, toutes les tables, fonctions SQL, triggers d'intégrité et de numérotation, **RLS complète**, bucket Storage + policies.
- Seed : admin Haris, **LSM Logistics** (`LSM`) et **Eurofret** (`EUR`), 2 contacts par organisation, ~10 tickets variés avec commentaires et entrées de temps.
- Auth : connexion, mot de passe oublié, définition du mot de passe, redirection selon le rôle, HashRouter.
- Tests pgTAP (liste ci-dessous).
- Workflow GitHub Actions de déploiement sur GitHub Pages.
- *Livrable : connexion fonctionnelle pour l'admin et un contact de chaque client, tests RLS verts.*

**Phase 2 — Cœur admin**
- Liste des tickets, détail en panneau latéral, changement statut / priorité / catégorie.
- Commentaires.
- Saisie du temps + moteur `src/lib/time/` avec tous ses tests Vitest.

**Phase 3 — Portail client et gestion des contacts**
- Mes tickets, nouveau ticket, détail, commentaires, temps visible via `get_ticket_time_entries()`.
- Réouverture automatique sur commentaire client.
- Écran Organisations & contacts, edge function `manage-contact` (invitation, désactivation), vérifiant que l'appelant est admin.

**Phase 4 — Pièces jointes**
- Upload web (bouton, glisser-déposer, coller une capture), validations type / taille, miniatures, téléchargement par URL signée.

**Phase 5 — Notifications sortantes** *(prérequis : domaine + compte fournisseur, section 16)*
- `email_outbox`, triggers, Database Webhook, edge function `send-email`, interface `EmailProvider` + implémentation Postmark, templates, relance pg_cron, préférences de notification.

**Phase 6 — Email entrant**
- Edge function `inbound-email` (section 8 complète), écran Quarantaine, tests Deno sur le parsing, le nettoyage du corps et le routage.

**Phase 7 — Export du temps**
- Écran Export, `computeMonthlyTimesheet`, `timesheetToCsv`, tests.

**Phase 8 — Finitions et mise en production**
- Recherche et filtres avancés, polish responsive et dark mode, états vides.
- MFA TOTP admin.
- Purge pg_cron des `raw_payload` de plus de 30 jours.
- CSP en balise `<meta>` (GitHub Pages ne permet pas les en-têtes HTTP).
- Checklist de mise en prod : signups désactivés, Site URL et Redirect URLs Auth limitées au domaine Pages, secrets en place, `supabase test db` rejoué sur la prod, test de bout en bout avec un compte de chaque organisation **avant d'inviter un vrai client**.

### Tests pgTAP minimaux (phase 1, enrichis à chaque phase)
- Un contact LSM ne voit **aucun** ticket, commentaire, pièce jointe ni fichier Storage d'Eurofret (et inversement).
- Un client ne peut pas lire `time_entries` directement ; `get_ticket_time_entries()` ne renvoie jamais `internal_note` et refuse un ticket d'une autre org.
- Un client ne peut pas modifier `status`, `priority`, `organization_id` ; ni le ticket d'un autre contact ; ni son propre `role` ou `organization_id`.
- Un client ne peut pas commenter un ticket fermé.
- Un contact désactivé ne voit plus rien.
- `email_outbox` et `inbound_emails` sont illisibles pour un client.
- `anon` ne voit rien.
- La numérotation produit `LSM-0001`, `LSM-0002`, `EUR-0001` de façon indépendante.

---

## 13. Exigences de sécurité transverses

- **Clé secrète Supabase jamais côté front ni dans le repo.** Les variables `VITE_*` sont publiques : URL et clé anon uniquement. La protection repose sur la RLS.
- RLS sur toute nouvelle table, **avec son test pgTAP dans le même lot**.
- Fonctions `SECURITY DEFINER` : `set search_path = ''`, noms qualifiés, `execute` révoqué pour `anon`.
- Aucun `dangerouslySetInnerHTML`. Tout contenu utilisateur et tout email entrant est rendu en texte.
- Edge functions : `manage-contact` vérifie le JWT **et** le rôle admin ; `inbound-email` et `send-email` vérifient leur secret.
- Fichiers : liste blanche de types côté bucket, noms nettoyés, URLs signées courtes.
- Données personnelles : purge des emails bruts à 30 jours, pas de contenu d'email dans les logs des edge functions au-delà du nécessaire.

---

## 14. Décisions par défaut (modifiables par Haris uniquement)

1. React 19 et Tailwind 4 (la v1 disait React 18) : versions stables actuelles, ciblées par shadcn/ui.
2. Postmark comme fournisseur email, derrière une interface interchangeable.
3. Temps comptabilisé = somme par ticket et par jour, arrondie au quart d'heure le plus proche, minimum 15 min.
4. Le client voit le temps **comptabilisé** et les notes publiques ; les notes internes jamais.
5. Tous les contacts d'une organisation voient tous ses tickets ; les notifications de réponse vont au créateur du ticket uniquement.
6. Expéditeur inconnu hors domaines autorisés → quarantaine, jamais de réponse automatique.
7. Pas de pièces jointes dans les emails sortants, uniquement un lien vers le portail.
8. Commentaire client sur un ticket en attente ou résolu → repasse en cours ; un ticket fermé n'accepte plus de commentaires.

---

## 15. Hors périmètre (ne pas implémenter)

- Facturation : devis, factures, taux horaires, PDF → futur module Argos Facturation.
- Collaborateurs internes / plusieurs admins.
- Commentaires internes (privés admin) sur les tickets.
- SLA, délais de réponse, fermeture automatique.
- Notifications push, Slack, Teams.
- Journal d'audit détaillé des modifications.
- Base de connaissances / FAQ client.

---

## 16. Prérequis côté Haris

**Avant la phase 1**
- Un projet Supabase (l'offre gratuite suffit pour démarrer).
- Un repo GitHub avec GitHub Pages activé.

**Avant les phases 5 et 6**
- Un nom de domaine.
- Un compte chez le fournisseur email.
- DNS : SPF, DKIM et DMARC pour le domaine d'envoi ; MX du sous-domaine de réception pointant vers le fournisseur.
- Secrets Supabase renseignés (section 3).
