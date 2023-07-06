# PaaSTech - Technical Architecture Document

## Contextualisation

### Description and intent

**_TODO: [UNIFIED WORK] write description of the software and its intent (what market needs it answers to)_**

### Technologies

**_TODO: [UNIFIED WORK] define the different technologies below, why they were chosen, and use arguments AND PROOFS to show research_**

#### Web application

**_TODO: [CLIENT] fill for web application frontend_**

#### CLI (Command-Line Interface)

**_TODO: [CLIENT] fill for the cli_**

#### Client API

**_TODO: [CLIENT] fill for the external API_**

#### Git controller

**_TODO: [GIT] fill for the git controller and architecture_**

#### Client applications deployment

**_TODO: [INFRA] fill for the architecture used to deploy clients' applications_**

#### Database

**_TODO: [CLIENT] fill for the unified database schema_**

#### CI/CD

**_TODO: [UNIFIED WORK] describe use of Github Actions as means of CI/CD_**

## Architecture

### Definitions

- the Service refers to PaaSTech as a whole;
- a Client is a user account created by an end user against the Service;
- a Project is a materialisation of a Git repository, created by a Client using either the web frontend or the CLI. A Project can be deployed by the Client by pushing its code to the Service.
- an Application (also referred to as Deployment) is an atomic unit of code, and is the result of a Project deployment. This unit is internally managed and can only be configured to a certain extent by the Client.

### Component interaction

```mermaid
graph LR

subgraph CLI/Website
A((CLI/Website))
end

subgraph Central API
B((Nest API))
end

subgraph Git Controller
C((Git Controller))
end

subgraph Pomegrenade
D((Pomegrenade))
end

A -->|REST API Request| B
B -->|REST API Response| A
B -->|gRPC Request| C
B -->|gRPC Request| D
C -->|gRPC Response| B
D -->|gRPC Response| B

```

**_TODO: continue the text and put the cli/website on the left of the schema_**

### Database architecture

**_TODO: [CLIENT] describe the database architecture, as referenced in the MCD in the README_**

### Detailed specification

#### Key constraints

**_TODO: [CLIENT, INFRA] the key constraints that should never be broken by the application (or at least the external parts, like the API and container exposition) in order to maintain security, isolation and client data safety_**

#### Client sign-up and login process

**_TODO: [CLIENT] mermaid diagram and description of the login process flow, for both the CLI and the web frontend_**

#### Projects storage

**_TODO: [GIT] how are the projects stored and how is authentication handled upon push_**

#### Client Applications

**_TODO: [INFRA] how are the Client applications provisioned, how they are exposed and how the underlying infrastructure is managed_**

## Post-mortem

### Organisational overview

**_TODO: [UNIFIED WORK] from an organisation standpoint, how was the entire team organised, how did the squads interact_**
