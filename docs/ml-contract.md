# Contrat de données du modèle de no-show

Ce document fixe ce que le modèle prédit, quand, et avec quelles informations.
Il protège le modèle contre la fuite de données (data leakage), étudiée en phase 8.
Version du contrat : v1 (modèle `noshow-v1`).

## 1. Objectif

- Question posée au modèle : "Quelle est la probabilité que cette cliente ne se présente pas à ce rendez-vous ?"
- Utilisation : badge de risque dans l'agenda du gérant (story G2) et bloc "risque no-show élevé" du dashboard.
- Le score aide le gérant à agir (envoyer un rappel, demander un acompte). Il ne bloque jamais une réservation et ne sert jamais à refuser une cliente.

## 2. La cible

- `is_no_show` = 1 si le statut final est `NO_SHOW`, 0 s'il est `COMPLETED`.
- Exclus de l'entraînement : les rendez-vous `CANCELLED` (créneau libéré, ce n'est pas une absence) et les rendez-vous futurs (`BOOKED` ou `CONFIRMED`, résultat inconnu).
- Même définition du no-show que dans `docs/kpi-dictionary.md`.

## 3. La date de décision

- Le score est calculé **48 h avant le rendez-vous** : `T = start_at - 48 h`.
- Si la cliente réserve moins de 48 h à l'avance, le score est calculé **à la réservation** : `T = booked_at`.
- Pourquoi 48 h : il reste assez de temps pour envoyer un rappel ou demander un acompte. Un score calculé trop tard ne permettrait plus d'agir.
- Règle d'or : à la date T, le modèle ne voit que ce qui est connu **avant T**. Tout ce qui arrive après T est interdit, même si cela arrive avant le rendez-vous.

Exemples :

| Réservation | Rendez-vous | Date de décision T |
|---|---|---|
| 1er novembre 10:00 | 10 novembre 15:00 | 8 novembre 15:00 |
| 9 novembre 18:00 | 10 novembre 15:00 | 9 novembre 18:00 (moins de 48 h : T = réservation) |

## 4. Variables autorisées

Une variable est autorisée si elle était connue à la date de décision.

| Variable | Type | Définition | Calcul | Connue à partir de |
|---|---|---|---|---|
| lead_time_hours | nombre | Avance avec laquelle la cliente a réservé. Plus le délai est long, plus le risque d'oubli ou de changement de plan augmente. | `start_at - booked_at` en heures | la réservation |
| hour_of_day | entier 0 à 23 | Heure du rendez-vous. Les créneaux très tôt ou en fin de journée ne sont pas manqués de la même façon. | issue de `start_at` (heure locale de Tunis) | la réservation |
| weekday | entier 1 à 7 | Jour de la semaine du rendez-vous. Le comportement diffère entre un lundi et un samedi. | issu de `start_at` (heure locale de Tunis) | la réservation |
| service_category | catégorie | Type de prestation (coupe, barbe, soin, etc.). Une prestation longue ou rare est réservée plus sérieusement. | issue de `services` | la réservation |
| price_tnd | nombre | Prix de la prestation en DT, figé à la réservation. Un prix élevé peut décourager l'absence. | copié sur le rendez-vous | la réservation |
| duration_min | nombre | Durée prévue de la prestation en minutes. | copiée sur le rendez-vous | la réservation |
| has_deposit | booléen | La cliente a-t-elle payé un acompte à la réservation ? Quelqu'un qui a déjà payé vient plus souvent. | `deposit_tnd > 0` à la réservation | la réservation |
| channel | catégorie | Canal de réservation : `WEB`, `PHONE` ou `WALK_IN` (sur place). Le canal reflète l'engagement de la cliente. | colonne `channel` | la réservation |
| is_first_visit | booléen | Est-ce la première visite de la cliente ? Sans historique, le risque est plus incertain. | aucun rendez-vous honoré avant T | calculé à T |
| prior_completed | entier | Nombre de rendez-vous honorés (`COMPLETED`) de la cliente avant T. | historique dont l'issue est connue avant T | calculé à T |
| prior_no_shows | entier | Nombre de no-show de la cliente avant T. C'est en général le signal le plus fort. | historique dont l'issue est connue avant T | calculé à T |
| prior_cancellations | entier | Nombre d'annulations de la cliente avant T. | historique dont l'issue est connue avant T | calculé à T |
| days_since_last_visit | nombre | Jours écoulés depuis la dernière visite honorée. Une cliente absente depuis longtemps est moins régulière. | jours entre la dernière visite honorée et T | calculé à T |

Précision sur les historiques : un rendez-vous passé compte seulement si son issue était déjà connue avant T. Exemple : un no-show survenu le 5 novembre compte pour un score calculé le 8 novembre, pas pour un score calculé le 1er novembre.

## 5. Variables interdites

Elles contiennent l'avenir par rapport à T.

| Variable | Pourquoi elle est interdite |
|---|---|
| status, cancelled_at | C'est le résultat que l'on cherche à prédire |
| reminder_replied | La réponse au rappel arrive après la date de décision |
| checked_in_at, paid_at | Ils arrivent le jour du rendez-vous |
| rating | L'avis est posté après la prestation |
| no_show_scores.* | Le score dépend du modèle lui-même |

Autres interdits : nom, téléphone, e-mail de la cliente (inutiles pour prédire et sensibles pour la vie privée), et toute variable sensible (origine, genre, religion) ou proxy évident.

Choix assumé : la confirmation du rappel est un signal fort, mais elle est connue après T. Pour la v1 on la sacrifie pour garder un modèle simple et sans fuite. Une v2 pourrait recalculer le score après la réponse (voir section 10).

## 6. Découpage des données

- Découpage chronologique, jamais aléatoire : 70 % des rendez-vous les plus anciens pour l'entraînement, 15 % suivants pour la validation, 15 % les plus récents pour le test.
- Le jeu de test sert une seule fois, pour le résultat final.
- Toutes les variables d'historique sont calculées à la date T de chaque rendez-vous, jamais avec toute la base.

## 7. Métriques et référence à battre

- Les no-show sont rares : l'exactitude (accuracy) est trompeuse et n'est pas retenue.
- Métriques principales : PR-AUC, et rappel (recall) et précision au seuil `HIGH`.
- Calibration : un score de 0,70 doit correspondre à environ 70 % d'absences observées.
- Référence à battre : règle simple "taux d'absence historique de la cliente" et taux global du salon. Le modèle n'est livré que s'il fait mieux sur le jeu de test.
- Les valeurs cibles et les seuils de risque (LOW, MEDIUM, HIGH) seront choisis en phase 8, avec une analyse de coût (une alerte inutile coûte un rappel, une absence ratée coûte un créneau perdu).

## 8. Sortie du service de scoring

```json
{
  "appointment_id": 123,
  "score": 0.55,
  "risk_level": "MEDIUM",
  "model_version": "noshow-v1",
  "scored_at": "2026-11-08T15:00:00+01:00",
  "top_factors": ["prior_no_shows", "lead_time_hours", "has_deposit"]
}
```

- `score` : nombre entre 0 et 1, stocké tel quel. L'interface peut l'afficher sur 100 (0,55 devient 55).
- `risk_level` : `LOW`, `MEDIUM` ou `HIGH`.
- `top_factors` : les trois facteurs principaux, traduits en phrases lisibles dans le panneau de détail de l'agenda.
- `model_version` et `scored_at` sont enregistrés avec le score pour l'audit.

## 9. Fiabilité et repli

- Si le service de scoring est indisponible, la réservation réussit quand même et le score reste vide. Le badge affiche "n/d".
- Un rendez-vous sans score est ignoré par les KPI qui utilisent le niveau de risque.

## 10. Suivi et évolution

- Le modèle est versionné (`noshow-v1`, `noshow-v2`, ...). Un changement de version est une décision documentée.
- Suivi mensuel : comparer les scores aux absences réelles. Si la calibration se dégrade, ré-entraîner.
- Évolution possible (v2) : scorer à plusieurs moments (réservation, 48 h, après la réponse au rappel). Cela exige de conserver des photos datées des variables à chaque instant (point-in-time correctness) et un jeu d'entraînement par moment. Écartée en v1 par choix de simplicité et de sécurité contre la fuite de données.