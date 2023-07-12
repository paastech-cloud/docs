# PaaSTech - Technical Architecture Document

## Contextualisation

### Description and intent

PaaSTech is a self-hostable PaaS (Platform as a Service), that is centred around offering the developer a simple deployment and operational experience.
Like every PaaS, the aim is to provide the users, in this case web developers, with an efficient and powerful way to package, deploy and expose their applications.

The main features are:

- a web UI to visualize and manage projects
- a CLI to push and deploy the applications
- easy configuration management for the deployed applications

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

***TODO: [CLIENT] fill for the unified database schema***

#### CI/CD

Each repository has a GitHub Actions CI to execute format check, linter and tests at each push. This ensures a good code quality and a good test coverage for our entire project.

Most projects use our own Actions available in a [central repository](https://github.com/paastech-cloud/.github). 
This ensures that all projects have the same CI/CD pipeline and that the CI/CD pipeline is easily maintainable.
Moreover, thanks to Rust easy documentation creation process, a simple CI job is able to build the documentation and deploy it on [GitHub Pages](https://pages.github.com/) for every Rust project.
Furthermore, at a tag push on main, or on a workflow call, a Docker image is built for every project that has this workflow set up.
This image is then pushed as a repository package to the [GitHub Container Registry](https://ghrc.io) and is available for use.

On top of that, the web UI is automatically deployed on [Github Pages](https://pages.github.com/).

#### Deployment

To deploy every application of PaaSTech on a single machine, a `docker-compose.yml` was made alongside a `setup.sh` script to set up the host machine easily and deploy the entire project.
As of today, PaaSTech is not scalable and is not meant to be deployed on multiple machines. 
Furthermore, even if the setup script does a bit of configuration, some folder are still needed to be created manually, the `.env` file needs to be filled and a [custom script](https://github.com/paastech-cloud/git-repo-manager/blob/main/hooks/post-receive) must be added in a specific directory on the host machine (`/srv/hooks` by default). 
This deployment method is available in [PaaSTech' `infra` repository](https://github.com/paastech-cloud/infra).
In this `docker-compose.yml`, we create 2 networks, one for the internal applications, and one for the client applications.
Each application is then deployed in a container, with environment variables that can be set up by the user in the `.env` file, and volumes as needed.
This file also ensure that:
- the API is actually available at [https://api.paastech.cloud](https://api.paastech.cloud)
- the git controller is available on the selected port
- all containers are started in order
- all containers can communicate with eachother
- Traefik is set up correctly.

## Architecture

### Definitions

- The Service refers to PaaSTech as a whole;
- A Client is a user account created by an end user against the Service;
- A Project is a materialisation of a Git repository, created by a Client using either the web frontend or the CLI. A Project can be deployed by the Client by pushing its code to the Service.
- An Application (also referred to as Deployment) is an atomic unit of code, and is the result of a Project deployment. This unit is internally managed and can only be configured to a certain extent by the Client.


### Component interaction

***TODO: [UNIFIED WORK] mermaid diagram of how components interact with eachother***

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

In the beginning, we held a few meetings including everyone to specify all of the necessary functionalities. Due to high number of people we weren't really able to be as productive as we wanted but managed to create an overall specifications chart.

Afterwards, the work was split up into 3 teams of roughly equal size:
 
- The `client/web api` team had the goal to create both frontends (a CLI and a small web ui) as well as building the REST API backend.
- The `git controller` team was responsible for handling client code through git and building [OCI images](https://opencontainers.org) out of it.
- The `infrastructure` team which was tasked with running images and exposing the client application to the user.

All teams were originally formed using a preferential voting system, before getting slightly revised once we realized the real amount of work in each team. 
This really improved our productivity, but nevertheless, we ran into multiple problems of miscommunication or lack of communication inside as well as between the different teams, slowing down our progress. After some time the communication got better and we managed to handle the links between each service. However, there is still progress to be made regarding this issue.
