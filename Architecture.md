# PaaSTech - Technical Architecture Document

## Contextualisation

### Description and intent

***TODO: [UNIFIED WORK] write description of the software and its intent (what market needs it answers to)***

### Technologies

Below are the technologies used for the different components of the application. We will discuss the choices that we made, and why the chosen technology fits in the ecosystem.

A list of TPMs (Third-Party Modules) might be written to give more insight about the component, but please keep in mind those lists are non-exhaustive and only the main TPMs are described.

***TODO: [UNIFIED WORK] define the different technologies below, why they were chosen, and use arguments AND PROOFS to show research***

#### Web application

***TODO: [CLIENT] fill for web application frontend***

#### CLI (Command-Line Interface)

***TODO: [CLIENT] fill for the cli***

#### Client API

The Client API, meaning the external API that Clients connect to and interact with, is made using [NestJS](https://nestjs.com/). 
Being a cutting-edge framework gaining more and more attention, NestJS is very versatile and stands 
out from the competition by proposing two things: 
first of all, a [list of support modules](https://www.npmjs.com/search?q=%40nestjs), which totals to 41 as of today;
then, an [extensive documentation with code examples](https://docs.nestjs.com/) ranging from basics to difficult recipes to setup.

NestJS' support modules provide a substantial support for the integration of well-known modules into the application. The main TPMs used for the API are:

- `class-validator`: validation of DTOs (Data Transfer Objects) and their properties before they are handed out to NestJS controllers
- `@nestjs/config`: ease of access to a config service, centralising both environment variables and `.env` files
- `passport` and `passport-jwt` (along with `@nestjs/passport` and `@nestjs/jwt`): layer of abstraction to handle different authentication methods, such as JWTs, OAuth2 or OIDC. Here, only the JWTs are used.
- `uuid`: generation of UUIDs v4, following [RFC 4122](https://www.rfc-editor.org/rfc/rfc4122)
- `nodemailer` with `handlebars`: sending of outbound emails through SMTP, with templating of HTML emails sent

Moreover, all of the support that NestJS provides doesn't make it slower. Benchmarks open-sourced in v7.6.13 ([available here](https://github.com/nestjs/nest/blob/e7fa96022e8b8580413490101683aabe387ca9b9/benchmarks/all_output.txt)) gave the following comparison:
- NestJS-express is nearly on par with Express for the average response time (`65.44 ms` versus `61.88 ms`)
- NestJS-express is twice as fast in maximum response time (only `325 ms` against Express' `747 ms`)
- NestJS-express can handle nearly twice as many requests concurrently, for the minimum amount doable (`14183 req/s` versus `8407 req/s`)
- NestJS-express is on par with Express for the number of requests handled concurrently, for the average amount doable (`15640` versus `16454.41`)

The API is currently running on NestJS-express v10.0.3, which just released. Two years have passed since the last benchmarks, and we can expect NestJS to be even faster as of now.

#### Database

***TODO: [CLIENT] fill for the unified database schema***

#### Git controller

***TODO: [GIT] fill for the git controller and architecture***

#### Client applications deployment

***TODO: [INFRA] fill for the architecture used to deploy clients' applications***

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
