# ADR 0003 — Désactivation logique plutôt que suppression

- Statut : accepté
- Contexte : un gérant peut vouloir retirer un coiffeur ou fermer un salon. Mais des rendez-vous passés y sont rattachés par clé étrangère, et ils alimentent le chiffre d'affaires, le taux de no-show et l'entraînement du modèle.
- Décision : on ne supprime jamais un salon, un coiffeur ou un service. On les désactive avec `active = false`. Un coiffeur désactivé ne reçoit plus de nouveaux rendez-vous (story GC-32), mais son historique reste lisible.
- Conséquences :
  - Avantage : l'historique et les KPI restent exacts, aucune clé étrangère ne casse, et on peut réactiver.
  - Inconvénient : les requêtes qui listent les coiffeurs ou créneaux disponibles doivent filtrer `active = true`. Un oubli ferait apparaître un coiffeur désactivé.
