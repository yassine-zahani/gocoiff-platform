# Séquence : réserver un créneau

Parcours d'une cliente qui réserve. Il couvre les critères C3 (créneau pris, réservation réussie) et S1 (la réservation survit à une panne du scoring).

```mermaid
sequenceDiagram
    actor Cliente
    participant Front as Angular
    participant API as Spring Boot
    participant DB as MySQL
    participant AI as AI service
    Cliente->>Front: choisit un créneau
    Front->>API: POST /api/appointments
    API->>DB: le créneau est-il libre pour ce coiffeur ?
    DB-->>API: libre ou pris
    alt créneau déjà pris
        API-->>Front: 409 Créneau indisponible
        Front-->>Cliente: message d'erreur
    else créneau libre
        API->>DB: INSERT rendez-vous (statut BOOKED, prix copié)
        Note over API,DB: si deux clientes réservent en même temps,<br/>la contrainte UNIQUE de MySQL refuse la seconde (erreur 1062)<br/>et l'API répond 409
        DB-->>API: rendez-vous enregistré
        API->>AI: POST /score (variables du modèle)
        opt le service de scoring répond
            AI-->>API: score, niveau de risque, version du modèle
            API->>DB: INSERT no_show_scores
        end
        API-->>Front: 201 rendez-vous créé
        Front-->>Cliente: confirmation
    end
```

Si l'AI service est en panne, le bloc `opt` est sauté : la réservation est quand même confirmée et le score est calculé plus tard.
