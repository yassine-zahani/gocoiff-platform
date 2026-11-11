# ADR 0002 — Montants en DECIMAL et prix figé sur le rendez-vous

- Statut : accepté
- Contexte : le dinar tunisien se divise en 1000 millimes, donc trois décimales. Les nombres à virgule flottante (FLOAT, DOUBLE) ne représentent pas exactement des valeurs comme 0,1 : sur des milliers de rendez-vous, un total peut dériver et mentir. Par ailleurs, le prix d'un service peut changer après une réservation.
- Décision :
  - tous les montants (prix, acompte) sont stockés en `DECIMAL(10,3)`, jamais en FLOAT ;
  - le prix est copié sur le rendez-vous au moment de la réservation (`appointments.price_tnd`) au lieu d'être relu dans `services`.
- Conséquences :
  - Avantage : les totaux sont exacts à la millime, et le chiffre d'affaires d'hier ne change pas quand le gérant modifie un tarif aujourd'hui.
  - Inconvénient : duplication volontaire de la donnée (le prix existe à deux endroits) et colonnes un peu plus lourdes. En Java on utilisera `BigDecimal`, pas `double`.
