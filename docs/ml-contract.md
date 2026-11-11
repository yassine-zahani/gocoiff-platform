# Contrat de données du modèle de no-show

Ce document fixe ce que le modèle prédit, avec quelles données, et comment on juge
qu'il est bon. Il lie l'équipe données (entraînement) et l'équipe application (API).
Toute modification passe par une revue.

## 1. Objectif

- Question posée au modèle : "Quelle est la probabilité que cette cliente ne se présente pas à ce rendez-vous ?"
- Utilisation : afficher un badge de risque dans l'agenda du gérant (story G2) et le bloc "risque no-show élevé" du dashboard.
- Le score aide le gérant à agir (rappel, confirmation). Il ne bloque jamais une réservation et ne sert jamais à refuser une cliente.

## 2. Moment de la prédiction

- Le score (story S1) est calculé à la création de la réservation.
- Il est recalculé quand une information change avant le rendez-vous (par exemple confirmation du rappel).
- Règle d'or : à chaque calcul, le modèle ne voit que ce qui est connu à cet instant précis. Rien de ce qui se passe le jour du rendez-vous.

## 3. Cible (label)

- Variable à prédire : `is_no_show`
  - 1 si le rendez-vous est `NO_SHOW`
  - 0 si le rendez-vous est `COMPLETED`
- Exclus de l'entraînement : `CANCELLED` (le créneau a été libéré, ce n'est pas une absence) et `CONFIRMED` (résultat inconnu).
- Cohérent avec le dictionnaire de KPI : même définition du no-show (`docs/kpi-dictionary.md`).

## 4. Variables d'entrée (features)

Toutes sont connues au moment de la prédiction.

| Variable | Type | Définition | Source |
|---|---|---|---|
| client_nb_visites_passees | entier | Rendez-vous `COMPLETED` ou `NO_SHOW` avant celui-ci | rendez-vous |
| client_nb_absences | entier | Rendez-vous `NO_SHOW` avant celui-ci | rendez-vous |
| client_taux_absence | décimal 0 à 1 | absences / visites passées, NULL si 0 visite | calculé |
| client_est_nouvelle | booléen | Aucune visite passée | calculé |
| client_nb_annulations | entier | Rendez-vous `CANCELLED` avant celui-ci | rendez-vous |
| delai_reservation_h | décimal | Heures entre la réservation et le rendez-vous | rendez-vous |
| jour_semaine | entier 1 à 7 | Jour du rendez-vous (heure locale de Tunis) | rendez-vous |
| heure_rdv | entier 0 à 23 | Heure du rendez-vous (heure locale de Tunis) | rendez-vous |
| duree_min | entier | Durée de la prestation | service |
| prix_dt | décimal | Prix figé sur le rendez-vous | rendez-vous |
| rappel_confirme | booléen | La cliente a confirmé le rappel | rendez-vous |
| salon_taux_noshow_90j | décimal 0 à 1 | Taux de no-show du salon sur les 90 jours avant la prédiction | calculé |

## 5. Variables interdites

- Fuite de données (data leakage) : tout ce qui n'est connu qu'après le rendez-vous (statut final, heure d'arrivée, paiement effectif, avis laissé).
- Identité directe : nom, téléphone, e-mail. Elles ne servent à rien pour prédire et posent un risque de vie privée.
- Aucune variable sensible (origine, genre, religion, etc.), ni de proxy évident.

## 6. Découpage des données

- Découpage chronologique, jamais aléatoire :
  - entraînement : 70 % des rendez-vous les plus anciens
  - validation : 15 % suivants
  - test : 15 % les plus récents
- Le jeu de test n'est utilisé qu'une fois, pour le résultat final.
- Les variables d'historique (nb d'absences, etc.) sont calculées "à la date du rendez-vous", pas avec toute la base.

## 7. Métriques et seuil d'acceptation

- Les no-show sont rares (classes déséquilibrées) : l'exactitude (accuracy) est trompeuse, on ne la retient pas.
- Métriques principales : PR-AUC, rappel (recall) et précision au seuil `HIGH`.
- Calibration : un score de 70 doit correspondre à environ 70 % d'absences observées.
- Référence à battre : règle simple "taux d'absence historique de la cliente" et taux global du salon. Le modèle n'est livré que s'il fait mieux que cette règle sur le jeu de test.
- Valeurs cibles à renseigner après les premiers essais (phase 8) : à renseigner en phase 8 (Machine Learning).

## 8. Sortie du service de scoring

```json
{
  "appointment_id": 123,
  "score": 55,
  "level": "MEDIUM",
  "model_version": "noshow-v1",
  "computed_at": "2026-11-11T15:00:00+01:00",
  "top_reasons": ["1 absence sur 3 visites", "rappel non confirmé"]
}
```

- `score` : entier de 0 à 100.
- `level` : `LOW` (moins de 30), `MEDIUM` (30 à 69), `HIGH` (70 et plus).
- `top_reasons` : 2 ou 3 raisons lisibles, affichées dans le panneau de détail de l'agenda.
- `model_version` et `computed_at` sont enregistrés avec le score pour pouvoir auditer et comparer les versions.

## 9. Fiabilité et repli

- Si le service de scoring est indisponible, la réservation réussit quand même et le score reste vide (NULL). Le badge affiche "n/d".
- Un rendez-vous sans score est exclu du KPI "taux de rendez-vous à risque élevé".

## 10. Suivi et versions

- Le modèle est versionné (`noshow-v1`, `noshow-v2`, ...). Le changement de version est une décision documentée.
- Suivi mensuel : comparer les scores prédits aux absences réelles. Si la calibration se dégrade, ré-entraîner.
