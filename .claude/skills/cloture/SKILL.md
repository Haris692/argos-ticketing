---
name: cloture
description: Clôture propre d'une session - état sauvegardé dans STATUS, commit, prochaine étape claire. À faire avant un /clear ou en fin de journée.
---

1. **Faire le point** en 8 lignes au maximum : tâches terminées pendant la session, tâche en cours (et où elle en est précisément), blocages, questions pour Haris.
2. **Confier au `scribe`** la mise à jour de STATUS (avec une « Prochaine étape » précise et actionnable), de la ROADMAP et des DECISIONS, puis le commit `chore(status): session <date>` si des fichiers ont changé.
3. **Tâche à moitié faite** : la noter dans STATUS, section « En cours », avec les fichiers touchés et la prochaine étape exacte. On ne commite jamais du code qui ne compile pas ou une migration qui ne passe pas `supabase db reset` : dans ce cas, `git stash` et mention du stash dans STATUS.
4. **Conclure auprès de Haris** : « État sauvegardé. Tu peux faire /clear. La prochaine fois : <action>. »
