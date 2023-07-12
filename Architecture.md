# PaaSTech - Technical Architecture Document

## Contextualisation

### Description and intent

***TODO: [UNIFIED WORK] write description of the software and its intent (what market needs it answers to)***

### Technologies

***TODO: [UNIFIED WORK] define the different technologies below, why they were chosen, and use arguments AND PROOFS to show research***

#### Web application

***TODO: [CLIENT] fill for web application frontend***

#### CLI (Command-Line Interface)

***TODO: [CLIENT] fill for the cli***

#### Client API

***TODO: [CLIENT] fill for the external API***

#### Git controller

***TODO: [GIT] fill for the git controller and architecture***

#### Client applications deployment

***TODO: [INFRA] fill for the architecture used to deploy clients' applications***

#### Database

The requirement for an ACID (Atomicity, Consistency, Isolation, and Durability) database led us to choose a relational database. For our application there wasn't a need to have a multitude of adaptable fields and the benefits of having consistent data with relationships between each other as well as having structured data were a more important factor.

The only flexible field is the environmental variables that are stored for each project, but since our chosen database, [PostgreSQL](https://www.postgresql.org/), supports the [JsonB](https://www.postgresql.org/docs/9.5/datatype-json.html) type, this could easily be implemented.

As market leader, PostgreSQL stands out for its performance and widespread use throughout the world. Not only is it free and open-source, it also allows also a larger set of extensions and features than other databases, like the support for more than 43 different [data types](https://www.postgresql.org/docs/current/datatype.html). Together with its flexibility and usage of [Multi-Version concurrency Control (MVCC)](https://www.postgresql.org/docs/current/glossary.html#GLOSSARY-CONCURRENCY) as well as our prior experiences made it more interesting that its major competitor [MySQL](https://www.mysql.com/).

The API connects to the database using an ORM called [Prisma](https://www.prisma.io/). Prisma also automates the generation and migration of the database as well as creating the classes associated to each table. Being an ORM that is both performant and allows the reorganisation of SQL data into simple classes, it was an important addition to our technology stack. Its complete documentation, active development and strong support made it more interesting than its competitors like [TypeORM](https://typeorm.io/) or [Objection JS](https://vincit.github.io/objection.js/).

#### CI/CD

***TODO: [UNIFIED WORK] describe use of Github Actions as means of CI/CD***


## Architecture

### Definitions

- the Service refers to PaaSTech as a whole;
- a Client (also referred to as a User) is a user account created by an end user against the Service;
- a Project is a materialisation of a Git repository, created by a Client using either the web frontend or the CLI. A Project can be deployed by the Client by pushing its code to the Service.
- an Application (also referred to as Deployment) is an atomic unit of code, and is the result of a Project deployment. This unit is internally managed and can only be configured to a certain extent by the Client.


### Component interaction

***TODO: [UNIFIED WORK] mermaid diagram of how components interact with eachother***

### Database architecture

As the database is unified and serves as a focal point of the application, everything is contained in the same logical database.
The architecture is as follows:

```mermaid
erDiagram
    USERS {
        uuid id PK
        varchar(40) username UK
        varchar(100) email UK
        varchar(255) password
        bool is_admin "Default false"
        uuid email_nonce UK "Nullable, default uuid()"
        uuid password_nonce UK "Nullable"
        timestamp created_at
        timestamp updated_at "Nullable"
    }
    SSH_KEYS {
        uuid id PK
        text value "public SSH key content"
        varchar(30) name
        timestamp created_at
        timestamp updated_at "Nullable"
        uuid user_id FK
    }
    PROJECTS {
        uuid id PK
        varchar(40) name
        jsonb config
        timestamp created_at
        timestamp updated_at "Nullable"
        uuid user_id FK
    }

    USERS ||--o{ PROJECTS : manage
    USERS ||--o{ SSH_KEYS : possess
```

As you can notice, every table has a UUID field as a primary key. Compared to [Serial](https://www.postgresql.org/docs/current/datatype-numeric.html) that is often used to identify a table, a UUID guarantees a better uniqueness across the whole database. 
To generate a [UUID](https://www.postgresql.org/docs/current/datatype-uuid.html), we are using the UUIDv4 standard, which generates each UUID randomly. Thus, there are roughly 103 trillion UUIDv4s, lowering the chance of finding a duplicate UUID to one-in-a-billion. 
Compared to them, Serials are limited to roughly 2 billion (2147483647) and will eventually clog up an dbe reduntant. 
However, they take up less space (4 bytes) than a UUID (16 bytes) but since it is a series of incrementing integers it offers information about the time of creation and makes it easier to guess the id whereas a UUID is generated randomly and is nearly impossible to duplicate.


Furthermore, since the projects will be deployed at a subdomain that is named after `{projectId}.user-app.paastech.cloud` we needed to hide the internal database structure that emerges with a serial id. Therefore, using a uuid prevents sharing sensitive data with everyone and prevents targeted attacks.


The table `users` contains all the necessary information about each user. Upon user creation, an `email_nonce` is automatically created and a user account is only considered active once the email has been confirmed and the field `email_nonce` is null.
Should the user wish to reset their password, a new UUID will be saved in the field `password_nonce`. To reset his password, the Client will need to provide said UUID together with the new password. Should another request to change the password been made before the password was reset, a new UUID will be generated and the old one will be overwritten. Once the password has been reset, `password_nonce` will be set to null. 
One user can have multiple projects or SSH keys.

The `ssh_keys` specified by each user allow them to push their repository onto our git server. Each SSH key can have a name to make it easier to distinguish multiple keys, however, it is not required.
An SSH key belongs to a user and not a project. Thus, it will allow the Client to access any of their repositories. Should a user possess more than one SSH key, each one will grant them access.


The `projects` table describes a project. Its field `config` contains all the environmental variables of the user, like database authentication. since the configuration and necessary variables change for every project we decided to store it as a flexible json field. We decided to use a jsonb field that stores the json data in binary form, allowing for better performances than a simple json field. 


### Detailed specification


#### Key constraints

***TODO: [CLIENT, INFRA] the key constraints that should never be broken by the application (or at least the external parts, like the API and container exposition) in order to maintain security, isolation and client data safety***

#### Client sign-up and login process

***TODO: [CLIENT] mermaid diagram and description of the login process flow, for both the CLI and the web frontend***

#### Projects storage

***TODO: [GIT] how are the projects stored and how is authentication handled upon push***

#### Client Applications

***TODO: [INFRA] how are the Client applications provisioned, how they are exposed and how the underlying infrastructure is managed***

## Post-mortem

### Organisational overview

***TODO: [UNIFIED WORK] from an organisation standpoint, how was the entire team organised, how did the squads interact***
