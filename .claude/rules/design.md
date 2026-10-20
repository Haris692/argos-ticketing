---
paths:
  - "src/styles/**"
  - "src/components/**"
  - "src/app/**"
  - "src/features/**/*.tsx"
---

# Design system « Apple » (spec §11)

- **Tokens** : couleurs uniquement via les variables CSS de `src/styles/` (fond, surface, surface élevée, texte, texte secondaire, séparateur, accent), définies en clair **et** en sombre ; thème clair / sombre / système.
- **Une seule couleur d'accent** (actions primaires, focus, sélection). Exception unique : le point rouge de la priorité urgente.
- **Statuts** en badges discrets ; jamais d'aplat saturé, de dégradé décoratif, d'ombre dure ni d'emoji.
- **Verre dépoli** seulement sur les éléments flottants (barre de navigation, panneau de détail, modales, menus).
- **Typo** : pile système, corps 15–16 px, jamais < 13 px, chiffres tabulaires pour références et durées.
- **Mouvement** : 150–250 ms, `cubic-bezier(0.25, 0.1, 0.25, 1)`, respect de `prefers-reduced-motion`.
- **Mobile** : tout utilisable à 375 px ; actions principales atteignables au pouce ; panneau de détail plein écran.
- **États vides** soignés : icône, phrase courte, action.
- **Vérification** : toute modification visuelle se vérifie en clair, en sombre et à 375 px ; ce qui ne l'a pas été va dans OUVERT.
