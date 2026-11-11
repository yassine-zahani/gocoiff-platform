# Modèle de données (base transactionnelle MySQL)

Dix tables. Les montants sont en `DECIMAL(10,3)` (ADR 0002), les dates en heure locale de Tunis (ADR 0001), et on désactive au lieu de supprimer (ADR 0003).

```mermaid
erDiagram
    USERS ||--o| CLIENTS : "a un profil"
    USERS ||--o{ SALONS : "possède"
    SALONS ||--o{ STAFF_MEMBERS : "emploie"
    SALONS ||--o{ SERVICES : "propose"
    STAFF_MEMBERS ||--o{ STAFF_SERVICES : "sait faire"
    SERVICES ||--o{ STAFF_SERVICES : "est assuré par"
    STAFF_MEMBERS ||--o{ WORKING_HOURS : "travaille selon"
    CLIENTS ||--o{ APPOINTMENTS : "réserve"
    SALONS ||--o{ APPOINTMENTS : "reçoit"
    STAFF_MEMBERS ||--o{ APPOINTMENTS : "assure"
    SERVICES ||--o{ APPOINTMENTS : "concerne"
    APPOINTMENTS ||--o| REVIEWS : "reçoit"
    APPOINTMENTS ||--o{ NO_SHOW_SCORES : "est évalué par"

    USERS {
        bigint id PK
        varchar email UK
        varchar password_hash
        varchar role "CLIENT, OWNER, ADMIN"
        datetime created_at
    }
    CLIENTS {
        bigint id PK
        bigint user_id FK, UK
        varchar first_name
        varchar last_name
        varchar phone
        varchar city
        datetime created_at
    }
    SALONS {
        bigint id PK
        bigint owner_user_id FK
        varchar name
        varchar city
        varchar governorate
        varchar address
        decimal latitude
        decimal longitude
        varchar category
        boolean active
        datetime created_at
    }
    STAFF_MEMBERS {
        bigint id PK
        bigint salon_id FK
        varchar display_name
        boolean active
    }
    SERVICES {
        bigint id PK
        bigint salon_id FK
        varchar name
        varchar category
        int duration_min "CHECK > 0"
        decimal price_tnd "DECIMAL(10,3), CHECK >= 0"
        boolean active
    }
    STAFF_SERVICES {
        bigint staff_id PK, FK
        bigint service_id PK, FK
    }
    WORKING_HOURS {
        bigint id PK
        bigint staff_id FK
        tinyint weekday "1 lundi à 7 dimanche"
        time start_time
        time end_time "CHECK > start_time"
    }
    APPOINTMENTS {
        bigint id PK
        bigint client_id FK
        bigint salon_id FK
        bigint staff_id FK
        bigint service_id FK
        datetime booked_at
        datetime start_at
        datetime end_at
        varchar status "BOOKED, CONFIRMED, COMPLETED, CANCELLED, NO_SHOW"
        decimal price_tnd "copie du prix à la réservation"
        decimal deposit_tnd
        varchar channel "WEB, PHONE, WALK_IN"
        datetime cancelled_at
        datetime updated_at "sert à l'ETL incrémental"
    }
    REVIEWS {
        bigint id PK
        bigint appointment_id FK, UK
        tinyint rating "1 à 5"
        varchar comment
        datetime created_at
    }
    NO_SHOW_SCORES {
        bigint id PK
        bigint appointment_id FK
        decimal score "entre 0 et 1"
        varchar risk_level "LOW, MEDIUM, HIGH"
        varchar model_version
        datetime scored_at
        json top_factors
    }
```

## Lecture des cardinalités

- `||--o{` : un à plusieurs. Un salon emploie zéro ou plusieurs coiffeurs ; un coiffeur appartient à un seul salon.
- `||--o|` : un à zéro-ou-un. Un compte a au plus un profil cliente ; un rendez-vous reçoit au plus un avis (clé unique sur `appointment_id`).
- `STAFF_SERVICES` est une table de jointure : elle transforme la relation plusieurs-à-plusieurs entre coiffeurs et services en deux relations un-à-plusieurs. Sa clé primaire est composée des deux clés étrangères.
- `NO_SHOW_SCORES` n'a pas de clé unique sur `appointment_id` : un rendez-vous peut être scoré plusieurs fois (nouvelle version du modèle, recalcul).

## Choix à retenir

- `appointments.salon_id` est redondant avec `staff_id` (un coiffeur appartient à un salon), mais il évite une jointure dans les requêtes de l'agenda et du dashboard.
- La colonne générée `blocking_start` (étape 2.5) n'apparaît pas ici : elle sert de garde-fou en base et ne fait pas partie du modèle métier.
