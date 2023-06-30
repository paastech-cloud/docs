# PaaSTech Cloud

## MCD

```mermaid
erDiagram
    USERS ||--o{ PROJECTS : own
    USERS ||--o{ SSH_KEYS : own
    PROJECTS ||--o{ DEPLOYMENTS : have

    USERS {
        integer id PK
        string username "UNIQUE"
        string email "UNIQUE"
        string password
        bool is_admin "DEFAULT FALSE"
        string email_nonce "NULLABLE"
        string password_nonce "NULLABLE"
        timestamp created_at
        timestamp updated_at "NULLABLE"
    }
    SSH_KEYS {
        integer id PK
        string value
        timestamp created_at
        timestamp updated_at "NULLABLE"
        integer user_id FK
    }
    PROJECTS {
        uuid uuid PK
        string name
        timestamp created_at
        timestamp updated_at "NULLABLE"
        integer user_id FK
    }
    DEPLOYMENTS {
        uuid uuid PK
        string name
        json config "Stores config such as env vars"
        timestamp created_at
        timestamp updated_at "NULLABLE"
        uuid project_uuid FK "References 'uuid' in table projects"
    }
```
