# Maquettes GoCoiff

Maquettes haute fidélité des écrans de l'application (16 écrans).
Figma : https://www.figma.com/design/WI17k4Xkt2LP5kxaWX4A5P/Landing-page---Go-Coiff?node-id=0-1&t=4Wt17NOwJRR6z6Yw-1

Charte : Powder Blue #84B4E6, Sweet Corn #FBF4DB, Spicy Orange #DD110A, police Inter.

## Écrans

| Fichier | Écran | Rôle | Story couverte |
|---|---|---|---|
| landing.png | Page d'accueil | Public | Point d'entrée vers C1 et G1 |
| inscription-client.png | Inscription cliente | Cliente | Accès aux stories C1 à C3 |
| connexion-client.png | Connexion cliente | Cliente | Accès aux stories C1 à C3 |
| recherche-salons.png | Recherche de salons | Cliente | C1 (GC-34) |
| page-salon-creneaux.png | Page salon et créneaux | Cliente | C2 (GC-35) |
| reservation-confirmee.png | Réservation confirmée | Cliente | C3 (GC-36) |
| mes-rdv-a-venir.png | Mes rendez-vous à venir | Cliente | C3 (GC-36), annulation gratuite jusqu'à 24 h |
| mes-rdv-passes.png | Mes rendez-vous passés | Cliente | C4 (GC-37), Should |
| inscription-gerant.png | Inscription gérant | Gérant | Accès aux stories G1 à G4 |
| connexion-gerant.png | Connexion gérant | Gérant | Accès aux stories G1 à G4 |
| mon-salon-services.png | Mon salon, services | Gérant | G1 (GC-38) |
| mon-salon-barbiers.png | Mon salon, barbiers | Gérant | G1 (GC-38) |
| mon-salon-horaires.png | Mon salon, horaires | Gérant | G1 (GC-38) |
| agenda-gerant.png | Agenda du jour | Gérant | G2 (GC-39), G3 (GC-40) |
| dashboard-gerant.png | Dashboard | Gérant | G4 (GC-41) |
| assistant-gerant.png | Assistant | Gérant | G5 (GC-42), Could |

## Stories Must sans écran

- S1 (GC-43), score de no-show : traitement serveur, sans écran propre. Son résultat s'affiche dans les badges de risque (Faible, Moyen, Élevé) de l'agenda et dans le bloc "risque no-show élevé" du dashboard.
- S2 (GC-44), copie nocturne vers l'entrepôt : traitement de nuit (ETL), invisible pour l'utilisateur.

Conclusion : toutes les stories Must utilisateur (C1, C2, C3, G1, G2, G3, G4) ont au moins un écran.