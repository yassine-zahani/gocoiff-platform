# Dictionnaire de KPI GoCoiff

Source de vérité unique pour tous les chiffres de la plateforme : requêtes SQL,
entrepôt de données, Power BI, dashboard Angular et assistant IA utilisent
exactement ces définitions.

## Conventions communes

- Statuts d'un rendez-vous : `CONFIRMED` (à venir), `COMPLETED`, `NO_SHOW`, `CANCELLED`.
- Une annulation n'est jamais un no-show : la cliente annule gratuitement jusqu'à 24 h avant, le créneau est libéré.
- Un rendez-vous est "clôturé" quand il est `COMPLETED` ou `NO_SHOW`.
- Montants en DT, type DECIMAL. Le prix est celui enregistré sur le rendez-vous au moment de la réservation, pas le prix actuel du service.
- Dates et heures en heure locale de Tunis. La période d'un KPI se rattache à la date du rendez-vous, pas à la date de création.
- Si le dénominateur vaut 0, le KPI vaut NULL (affiché "n/d"), jamais 0.
- Les rendez-vous sont comptés par salon. L'admin (A1) agrège sur tous les salons.

## 1. Chiffre d'affaires
- Définition : somme des prix des prestations effectivement réalisées.
- Formule : somme du prix des rendez-vous `COMPLETED` de la période.
- Unité : DT
- Période : jour, semaine, mois (dashboard : 14 derniers jours).
- Source : table des rendez-vous (prix, statut, date) du salon.
- Cas limites : les no-show et les annulations ne rapportent rien. Les rendez-vous à venir ne comptent pas (ils sont "prévus", pas encaissés).

## 2. Taux d'occupation
- Définition : part du temps disponible des barbiers réellement réservée.
- Formule : minutes réservées (`CONFIRMED` + `COMPLETED` + `NO_SHOW`) / minutes ouvertes (horaires du salon x barbiers actifs).
- Unité : %
- Période : jour, semaine, mois.
- Source : rendez-vous (durée, statut), horaires du salon, barbiers.
- Cas limites : un no-show occupe le créneau, donc il compte au numérateur. Un barbier désactivé ne compte plus dans les minutes ouvertes à partir de sa désactivation. Jour de fermeture : dénominateur 0, donc NULL.

## 3. Taux de no-show
- Définition : part des rendez-vous clôturés où la cliente ne s'est pas présentée.
- Formule : `NO_SHOW` / (`COMPLETED` + `NO_SHOW`).
- Unité : %
- Période : semaine, mois.
- Source : rendez-vous (statut, date).
- Cas limites : les `CANCELLED` sont exclus du numérateur et du dénominateur (créneau libéré, pas d'absence). Les `CONFIRMED` sont exclus (le résultat n'est pas connu). Un rendez-vous passé resté `CONFIRMED` n'est pas compté tant que le gérant ne l'a pas clôturé : c'est la raison du compteur "à marquer" de l'agenda.

## 4. Taux de rétention
- Définition : part des clientes de la période qui étaient déjà venues avant.
- Formule : clientes ayant au moins un `COMPLETED` dans la période ET au moins un `COMPLETED` avant la période / clientes ayant au moins un `COMPLETED` dans la période.
- Unité : %
- Période : mois.
- Source : rendez-vous (cliente, statut, date).
- Cas limites : une cliente qui vient 2 fois dans la même période mais jamais avant est nouvelle, pas fidèle. Première période d'un salon : NULL.

## 5. Nombre de rendez-vous
- Définition : volume de rendez-vous pris sur la période, hors annulations.
- Formule : nombre de rendez-vous `CONFIRMED` + `COMPLETED` + `NO_SHOW`.
- Unité : nombre
- Période : jour, semaine, mois.
- Source : rendez-vous.
- Cas limites : un rendez-vous reprogrammé est une annulation plus une nouvelle réservation, il compte une seule fois.

## 6. Panier moyen
- Définition : montant moyen d'un rendez-vous réalisé.
- Formule : chiffre d'affaires / nombre de `COMPLETED`.
- Unité : DT
- Période : semaine, mois.
- Source : rendez-vous.
- Cas limites : aucun `COMPLETED` : NULL.

## 7. Taux d'annulation
- Définition : part des rendez-vous pris qui ont été annulés par la cliente.
- Formule : `CANCELLED` / (`CONFIRMED` + `COMPLETED` + `NO_SHOW` + `CANCELLED`).
- Unité : %
- Période : semaine, mois.
- Source : rendez-vous (statut, date).
- Cas limites : seules les annulations faites dans la règle des 24 h existent dans les données (l'application refuse les autres).

## 8. Taux de rendez-vous à risque élevé
- Définition : part des rendez-vous à venir dont le score de no-show est dans la catégorie `HIGH`.
- Formule : rendez-vous `CONFIRMED` avec niveau `HIGH` / rendez-vous `CONFIRMED`.
- Unité : %
- Période : instantané (agenda du jour, jours à venir).
- Source : rendez-vous (statut), scores de no-show (niveau).
- Cas limites : rendez-vous sans score calculé : exclus du numérateur et du dénominateur. Seuils des niveaux (indicatif : LOW moins de 30, MEDIUM de 30 à 69, HIGH 70 et plus) : à figer dans `docs/ml-contract.md`.

## 9. Note moyenne des avis
- Définition : moyenne des notes laissées par les clientes après une prestation.
- Formule : moyenne des notes (1 à 5) des avis du salon.
- Unité : note sur 5, une décimale.
- Période : 90 derniers jours et global.
- Source : avis, liés à un rendez-vous `COMPLETED`.
- Cas limites : aucun avis : NULL. Un avis n'est possible que sur un rendez-vous `COMPLETED` (story C4, priorité Should).