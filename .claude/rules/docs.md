---
paths:
  - "docs/**"
---

# Règles de la documentation

- En français, concise et à jour : on remplace ce qui est devenu faux au lieu d'empiler.
- **Tailles maximales** : STATUS 60 lignes ; plan de phase 150 lignes ; DECISIONS une ligne par décision.
- **La spec** (`docs/ARGOS-TICKETING-SPEC.md`) appartient à Haris : on ne la modifie qu'avec son accord explicite, et chaque changement s'accompagne d'une ligne dans DECISIONS.
- **Statuts de décision** : VALIDÉ (décidé par Haris), PROPOSÉ (choix par défaut de l'équipe), À DÉCIDER. Rien ne passe en VALIDÉ sans l'accord explicite de Haris.
- **Titres de section** au format `## n. Titre` (spec) ou `## Px — Titre` (ROADMAP), et on n'y touche pas : les agents les retrouvent avec `grep`.
