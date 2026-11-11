# ADR 0001 — Heure locale de Tunis

- Statut : accepté
- Contexte : tous les salons sont en Tunisie (UTC+1, sans changement d'heure). Gérer des fuseaux horaires ajouterait de la complexité sans bénéfice aujourd'hui.
- Décision : stocker toutes les dates et heures en heure locale de Tunis, sans fuseau. La base MySQL est configurée à `+01:00` et les variables du modèle (`hour_of_day`, `weekday`) se lisent dans cette heure.
- Conséquences :
  - Avantage : simple partout (base, API, KPI, modèle de no-show), aucun décalage à gérer.
  - Inconvénient : à refaire si GoCoiff s'ouvre à d'autres pays ou fuseaux, car les anciennes dates n'auraient pas d'information de fuseau.
