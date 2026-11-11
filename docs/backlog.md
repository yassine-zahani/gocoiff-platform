Backlog produit GoCoiff

Priorités : MoSCoW (Must, Should, Could). Critères : Étant donné / Quand / Alors. Toutes les heures sont en heure locale de Tunis (voir ADR 0001).

Vue d'ensemble
ID	Ticket	En tant que	Je veux	Priorité
C1	GC-34	Cliente	rechercher des salons par ville et par service	Must
C2	GC-35	Cliente	voir les créneaux libres d'un coiffeur pour un service	Must
C3	GC-36	Cliente	réserver et annuler un rendez-vous	Must
C4	GC-37	Cliente	laisser un avis après la prestation	Should
G1	GC-38	Gérant	gérer mes services, mes coiffeurs et leurs horaires	Must
G2	GC-39	Gérant	voir l'agenda du jour avec le risque de no-show de chaque rendez-vous	Must
G3	GC-40	Gérant	marquer un rendez-vous comme terminé ou no-show	Must
G4	GC-41	Gérant	consulter un dashboard : chiffre d'affaires, occupation, no-show, rétention	Must
G5	GC-42	Gérant	poser une question sur mes chiffres en langage naturel	Could
S1	GC-43	Système	calculer un score de no-show à chaque réservation	Must
S2	GC-44	Système	copier chaque nuit les données vers l'entrepôt	Must
A1	GC-45	Admin	voir les indicateurs de toute la plateforme	Could
Critères d'acceptation des stories Must
C1 - Rechercher des salons (GC-34)
Étant donné que le salon "Chez Sami" est actif, situé à Tunis et propose une coupe homme, quand une cliente cherche "Tunis" avec le service "coupe homme", alors le salon apparaît dans les résultats.
Étant donné que le salon "Chez Sami" est désactivé (active = false), quand une cliente cherche "Tunis" avec le service "coupe homme", alors le salon n'apparaît pas dans les résultats.
Étant donné qu'aucun salon de Sfax ne propose de balayage, quand une cliente cherche "Sfax" avec le service "balayage", alors la liste est vide et un message "Aucun salon trouvé" s'affiche.
C2 - Voir les créneaux libres (GC-35)
Étant donné que Sami travaille de 9 h à 18 h le lundi et qu'il a un rendez-vous de 10 h à 11 h, quand une cliente demande les créneaux pour une coupe de 30 min ce lundi, alors aucun créneau ne chevauche 10 h à 11 h.
Étant donné qu'un rendez-vous de Sami est annulé, quand une cliente rafraîchit les créneaux, alors le créneau libéré réapparaît.
Étant donné que le coiffeur est désactivé, quand une cliente demande ses créneaux, alors la liste est vide.
C3 - Réserver et annuler (GC-36)
Étant donné qu'un créneau de 10 h est libre pour Sami, quand une cliente le réserve, alors le rendez-vous passe au statut BOOKED, le prix du service est copié dans le rendez-vous et le créneau disparaît de la liste des créneaux libres.
Étant donné qu'un créneau vient d'être réservé par une autre cliente, quand une seconde cliente tente de le réserver, alors la réservation est refusée avec un message "Créneau indisponible".
Étant donné un rendez-vous BOOKED dans 48 h, quand la cliente l'annule, alors le statut devient CANCELLED, la date d'annulation est enregistrée et le créneau est de nouveau libre.
Étant donné un rendez-vous BOOKED dans 5 h, quand la cliente tente de l'annuler en ligne, alors l'annulation est refusée avec le message "Annulation gratuite possible jusqu'à 24 h avant : contactez le salon", et le statut reste BOOKED.
Étant donné un rendez-vous BOOKED qui commence dans exactement 24 h, quand la cliente l'annule, alors l'annulation est acceptée. Décision : "jusqu'à 24 h" est inclusif, car une règle ambiguë à la limite crée des litiges. À 23 h 59 min, elle est refusée.
G1 - Gérer services, coiffeurs, horaires (GC-38)
Étant donné un gérant connecté, quand il ajoute le service "Brushing" de 45 min à 25 TND, alors le service apparaît dans son salon avec ces valeurs.
Étant donné un coiffeur qui a des rendez-vous passés, quand le gérant le désactive, alors il n'est plus proposé aux nouvelles réservations et ses anciens rendez-vous restent visibles.
Étant donné un gérant, quand il enregistre un horaire dont l'heure de fin est avant l'heure de début, alors l'enregistrement est refusé.
G2 - Agenda du jour avec risque de no-show (GC-39)
Étant donné trois rendez-vous aujourd'hui, quand le gérant ouvre son agenda, alors ils sont triés par heure et chacun affiche un niveau de risque Faible, Moyen ou Élevé.
Étant donné que le service de scoring est indisponible, quand le gérant ouvre son agenda, alors les rendez-vous s'affichent quand même, avec le risque "Non calculé".
G3 - Terminer ou marquer no-show (GC-40)
Étant donné un rendez-vous BOOKED dont l'heure est passée, quand le gérant le marque no-show, alors le statut devient NO_SHOW et ne peut plus changer.
Étant donné un rendez-vous BOOKED dont l'heure est passée, quand le gérant le marque terminé, alors le statut devient COMPLETED.
Étant donné un rendez-vous CANCELLED, quand le gérant tente de le marquer no-show, alors l'action est refusée.
Étant donné un rendez-vous dans le futur, quand le gérant tente de le marquer no-show, alors l'action est refusée.
G4 - Dashboard (GC-41)
Étant donné cinq rendez-vous du mois (3 COMPLETED à 40 TND, 1 NO_SHOW à 40 TND, 1 CANCELLED), quand le gérant ouvre le dashboard, alors le chiffre d'affaires affiche 120 TND, le manque à gagner 40 TND et le taux de no-show 25 %.
Étant donné une période sans aucun rendez-vous, quand le gérant ouvre le dashboard, alors les KPI affichent 0 ou "n/a" sans erreur de division par zéro.
S1 - Score de no-show (GC-43)
Étant donné une nouvelle réservation, quand elle est enregistrée, alors un score entre 0 et 1, un niveau de risque et la version du modèle sont stockés pour ce rendez-vous.
Étant donné que le service de scoring répond en erreur, quand une cliente réserve, alors la réservation réussit quand même et le score est calculé plus tard.
S2 - Copie nocturne vers l'entrepôt (GC-44)
Étant donné des rendez-vous modifiés depuis la dernière exécution, quand l'ETL s'exécute à 2 h du matin, alors ces lignes sont présentes dans l'entrepôt avec leur nouveau statut.
Étant donné que l'ETL est lancé deux fois de suite sans nouvelle donnée, quand la seconde exécution se termine, alors l'entrepôt contient le même nombre de lignes (aucun doublon).
Étant donné que MySQL est injoignable, quand l'ETL s'exécute, alors il échoue avec une erreur claire et n'écrit aucune donnée partielle.