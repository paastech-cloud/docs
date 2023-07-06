# PaaSTech - Technical Architecture Document

## Contextualisation

### Description and intent

***TODO: [UNIFIED WORK] write description of the software and its intent (what market needs it answers to)***

### Technologies

In the deployment manager, Pomegranate, the following technologies are used:

- The Rust programming language, for developing the service. We used this programming language because
  we wanted a performant language that could be trusted with this job, and with support for the other
  technologies we use. Today, we know that this was not the best choice as it might slow us down
  during development, and is not a first-party choice for libraries such as Docker and Kubernetes that
  we will use later on.
- Docker, for running the containers that contain the client applications. We used this because we already
  know how to use it, and it is straightforward to communicate with the Docker server to manage containers.
  It also handles for us networking and storage, so we don't have to worry about those issues.
- gRPC for allowing other services to communicate with us. This is the go-to framework for internal
  communications between services in this project.
- Traefik as the ingress controller. It is used because it discovers when containers are started and stopped,
  and can dynamically reconfigure itself to create and delete routes as those events happen.

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

***TODO: [CLIENT] fill for the unified database schema***

#### CI/CD

***TODO: [UNIFIED WORK] describe use of Github Actions as means of CI/CD***


## Architecture

### Definitions

- the Service refers to PaaSTech as a whole;
- a Client is a user account created by an end user against the Service;
- a Project is a materialisation of a Git repository, created by a Client using either the web frontend or the CLI. A Project can be deployed by the Client by pushing its code to the Service.
- an Application (also referred to as Deployment) is an atomic unit of code, and is the result of a Project deployment. This unit is internally managed and can only be configured to a certain extent by the Client.


### Component interaction

***TODO: [UNIFIED WORK] mermaid diagram of how components interact with eachother***

```mermaid
flowchart LR

git<-.git over ssh.->client["CLI"]

git["Git server"]<--grpc-->api

client<--http-->api

api["API"]<--grpc-->pomegranate["Pomegranate"]

pomegranate<-.docker.sock.->docker(["Docker"])

api<--TCP/IP-->id1[(Database)]
```

### Database architecture

***TODO: [CLIENT] describe the database architecture, as referenced in the MCD in the README***

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
