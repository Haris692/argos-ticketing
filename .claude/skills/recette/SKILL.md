---
name: recette
description: Prépare la checklist de recette d'une phase pour Haris, ou intègre son retour (bugs, ajustements, décisions).
argument-hint: "[Px | retour]"
---

Argument : « $ARGUMENTS »

## Préparer une recette (argument = Px)

Écris `docs/phases/Px-recette.md`, et donne à Haris un message de 25 lignes au maximum :
1. **Comment lancer** : commandes (`docs/OUTILS.md`), URL locale, comptes de test du seed (renvoyer vers `supabase/seed.sql`, ne pas recopier les mots de passe dans le message).
2. **Ce que Haris doit configurer** (Supabase, GitHub, DNS, Postmark…), s'il y a quelque chose : étapes précises.
3. **5 à 8 choses à tester**, concrètes. Par exemple : « Connecte-toi avec un contact LSM, puis ouvre l'URL `#/tickets/EUR-0001` : tu dois voir une erreur, pas le ticket. »
4. **Toujours** : rendu en clair **et** en sombre, et à 375 px de large.
5. **3 à 5 questions de ressenti**, notées de 1 à 5 avec un commentaire libre (clarté, rapidité, direction « Apple » de la spec §11).
6. **Ce qui est volontairement absent** (phases suivantes, spec §15), pour éviter les retours hors sujet.

Ensuite, fais ajouter « Recette en attente : Px » dans STATUS par le `scribe`.

## Intégrer un retour (argument = retour)

Commence par résumer ce que tu as compris en 8 lignes au maximum, puis transforme le retour de Haris en actions :
- **Bugs** : de nouvelles tâches `Tx.y-fix`, à exécuter avec `/tache`.
- **Ajustements d'interface** : tâches pour `dev-front`.
- **Décisions** : à faire consigner par le `scribe` dans DECISIONS.
- **Validation** : si Haris valide la phase, le `scribe` passe STATUS à « Recette validée : Px » et pose le tag `px`.

Pour tout changement de la **spec** (règle métier, décision §14), demande confirmation à Haris avant de l'appliquer.
