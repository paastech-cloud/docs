# PaaSTech Cloud

## MCD

```mermaid
erDiagram
    USERS ||--o{ PROJECTS : own
    USERS ||--o{ SSH_KEYS : own
    DEPLOYMENTS ||--o{ PROJECTS : contain
    USERS {
        integer id PK
        string username "UNIQUE"
        string email "UNIQUE"
        string password
        bool is_verified
        string email_nonce "NULLABLE"
        timestamp created_at
        timestamp updated_at
    }
    SSH_KEYS {
        integer id PK
        string value
        timestamp created_at
        timestamp updated_at
        integer user_id FK
    }
    PROJECTS {
        string uuid "UNIQUE"
        string name
        timestamp created_at
        timestamp updated_at
        integer user_id FK
    }
    DEPLOYMENTS {
        string uuid PK "UNIQUE"
        string name
        bson config "Stores config such as env vars"
        timestamp created_at
        timestamp updated_at
        integer project_uuid FK
    }
```