# PaaSTech - Technical Architecture Document

## Contextualisation

### Description and intent

***TODO: [UNIFIED WORK] write description of the software and its intent (what market needs it answers to)***

### Technologies

#### Application manager

In the application manager, Pomegranate, the following technologies were used:

- The Rust programming language, for developing the service. We used this programming language because
  we wanted a performant language that could be trusted with this job, and with support for the other
  technologies we use. Today, we know that this was not the best choice as it might slow us down
  during development, and is not a first-party choice for libraries such as Docker and Kubernetes that
  we will use later on.
- Docker, for running the containers that contain the client applications. We chose this because we already
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

In this iteration, we decided to use Docker as the orchestrator for the client applications. 
This choice helped us to quickly set up the MVP and get a working product as soon as possible.

However, we are aware that this choice is not the best one for the long term, as it is not greatly scalable nor very secure.
As a matter of fact, we will implement Kubernetes as the orchestrator for the client applications in the next iteration.
We also think of using MicroVM as a more secure alternative to containers. It is also easily integrable with Kubernetes through Firecracker.

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

api<-->front["Web frontend"]

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

Pomegranate, our application manager, is responsible for starting, stopping and basically interacting with the client applications.
It is also responsible for managing the networking of the applications.

For this iteration, we used Docker as the execution engine. Through [Bollard](https://crates.io/crates/bollard/), a rust crate, we can communicate with the Docker daemon through its API.
Bollard is a quite powerful crate, and allows us to do everything we need with Docker, such as:
- Starting and stopping containers,
- Creating networks,
- Creating volumes,
- Managing images,
- Fetching logs,
- Fetching stats

Pomegranate is completely stateless, and the API is the only external way to interact with it. 
Once a container is started, Pomegranate does not keep track of it, and it is the responsibility of the execution engine to manage it.

To expose newly spawned containers to the World Wide Web, [Traefik](https://doc.traefik.io/traefik/) is configured to answer on 80 and 443 ports.
Then a set of labels is attributed to each container to create a unique subdomain `<app_uuid>.user-app.<fqdn>` redirecting to the port 80 of the associated app.

TLS Termination is handled by Traefik, by resolving the DNS-O1 challenge with the TLS provider of choice. This allows to have a valid certificate for all subdomains of PaasTech.
In this case, the provider is Porkbun.



## Post-mortem

### Organisational overview

***TODO: [UNIFIED WORK] from an organisation standpoint, how was the entire team organised, how did the squads interact***
