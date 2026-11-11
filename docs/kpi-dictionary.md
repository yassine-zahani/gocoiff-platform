# Dictionnaire de KPI GoCoiff

Source unique de vérité pour tous les chiffres de la plateforme : requêtes SQL,
entrepôt de données, Power BI, dashboard Angular et assistant IA utilisent
exactement ces définitions.

## Conventions communes

- Statuts d'un rendez-vous : `BOOKED` puis `CONFIRMED` (à venir), et les états finaux `COMPLETED`, `NO_SHOW`, `CANCELLED`.
- Un rendez-vous est "honoré" quand il est `COMPLETED`.
- Une annulation n'est jamais un no-show : la cliente annule gratuitement jusqu'à 24 h avant, le créneau est libéré.
- Montants en DT, type DECIMAL(10,3) (1 DT = 1000 millimes). Le prix utilisé est `price_tnd`, copié sur le rendez-vous à la réservation, pas le prix actuel du service.
- Heures en heure locale de Tunis. La période d'un KPI se rattache à `start_at` (date du rendez-vous).
- Si le dénominateur vaut 0, le KPI vaut NULL (affiché "n/d"), jamais 0.
- Les KPI se calculent par salon. L'admin (A1) agrège sur tous les salons.

Exemple commun : sauf mention contraire, les exemples chiffrés utilisent ces 5 rendez-vous inventés, pour un même salon et une même journée.

| Rdv | Statut | price_tnd | duration_min | booked_at puis start_at (délai) |
|---|---|---|---|---|
| R1 | COMPLETED | 20.000 | 30 | 48 h avant |
| R2 | COMPLETED | 35.000 | 45 | 24 h avant |
| R3 | COMPLETED | 30.000 | 30 | 72 h avant |
| R4 | NO_SHOW | 25.000 | 30 | 6 h avant |
| R5 | CANCELLED | 20.000 | 30 | 10 h avant |

Minutes de travail disponibles ce jour-là (horaires des barbiers) : 600.

## 1. Chiffre d'affaires
- Définition : argent gagné sur les prestations réalisées.
- Formule : somme de `price_tnd` des rendez-vous `COMPLETED`.
- Unité : DT. Période : jour, semaine, mois (dashboard : 14 derniers jours).
- Source : `appointments` (price_tnd, status, start_at).
- Cas limites : un rendez-vous seulement réservé ne compte pas. Les no-show et annulations ne rapportent rien.
- Exemple : R1 + R2 + R3 = 20 + 35 + 30 = **85.000 DT**.

## 2. Manque à gagner
- Définition : argent perdu à cause des no-show. KPI central du projet.
- Formule : somme de `price_tnd` des rendez-vous `NO_SHOW`.
- Unité : DT. Période : semaine, mois.
- Source : `appointments`.
- Cas limites : les annulations ne comptent pas (le créneau a pu être revendu).
- Exemple : R4 = **25.000 DT**.

## 3. Panier moyen
- Définition : dépense moyenne par rendez-vous honoré.
- Formule : chiffre d'affaires ÷ nombre de `COMPLETED`.
- Unité : DT. Période : semaine, mois.
- Source : `appointments`.
- Cas limites : aucun `COMPLETED` : NULL.
- Exemple : 85 ÷ 3 = **28.333 DT**.

## 4. Taux de no-show
- Définition : part des rendez-vous manqués parmi ceux qui devaient avoir lieu.
- Formule : `NO_SHOW` ÷ (`COMPLETED` + `NO_SHOW`).
- Unité : %. Période : semaine, mois.
- Source : `appointments`.
- Cas limites : annulations et rendez-vous futurs exclus du numérateur et du dénominateur. Un rendez-vous passé non clôturé n'est pas compté tant que le gérant ne l'a pas marqué (compteur "à marquer" de l'agenda).
- Exemple : 1 ÷ (3 + 1) = **25 %**.

## 5. Taux d'annulation
- Définition : part des rendez-vous annulés.
- Formule : `CANCELLED` ÷ tous les rendez-vous de la période.
- Unité : %. Période : semaine, mois.
- Source : `appointments` (status, cancelled_at).
- Cas limites : les annulations tardives pourront être séparées plus tard (défi du TP).
- Exemple : 1 ÷ 5 = **20 %**.

## 6. Taux d'occupation
- Définition : part du temps de travail qui est réservé.
- Formule : minutes réservées (hors annulés) ÷ minutes des horaires de travail.
- Unité : %. Période : jour, semaine, mois.
- Source : `appointments` (duration_min), `working_hours`, `staff_members` actifs.
- Cas limites : jour de fermeture : NULL. Variante "occupation réelle" : même formule avec les seuls `COMPLETED`. L'écart entre les deux vaut les no-show.
- Exemple : réservées hors annulé = 30 + 45 + 30 + 30 = 135 min, donc 135 ÷ 600 = **22,5 %**. Occupation réelle : 105 ÷ 600 = **17,5 %**. Écart : 5 points = les 30 min du no-show R4.

## 7. Rétention à 60 jours
- Définition : part des clientes qui reviennent.
- Formule : clientes avec un 2e rendez-vous honoré dans les 60 jours après le 1er ÷ clientes avec un 1er rendez-vous honoré dans la période.
- Unité : %. Période : mois (cohorte).
- Source : `appointments` (client_id, status, start_at).
- Cas limites : demande une fenêtre de 60 jours complète après la période, sinon NULL.
- Exemple (autre jeu) : 5 clientes ont leur 1er rendez-vous honoré en septembre. A et B reviennent dans les 60 jours, C, D et E non. Résultat : 2 ÷ 5 = **40 %**.

## 8. Délai de réservation
- Définition : avance moyenne de réservation. Variable clé du modèle.
- Formule : moyenne de (`start_at` moins `booked_at`) en heures.
- Unité : heures. Période : semaine, mois.
- Source : `appointments`.
- Cas limites : on inclut tous les statuts (le délai existe même pour une annulation).
- Exemple : (48 + 24 + 72 + 6 + 10) ÷ 5 = 160 ÷ 5 = **32 h**.

## 9. Note moyenne
- Définition : satisfaction des clientes.
- Formule : moyenne de `rating` (1 à 5).
- Unité : note sur 5. Période : 90 jours et global.
- Source : `reviews`, liés à un rendez-vous `COMPLETED`.
- Cas limites : uniquement sur les avis déposés. Aucun avis : NULL.
- Exemple : R1, R2, R3 laissent 5, 4 et 3. R4 et R5 n'ont pas d'avis (ils ne comptent pas). Résultat : 12 ÷ 3 = **4,0**.