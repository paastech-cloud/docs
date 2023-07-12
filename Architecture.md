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

Below are the technologies used for the different components of the application. We will discuss the choices that we made, and why the chosen technology fits in the ecosystem.

A list of TPMs (Third-Party Modules) might be written to give more insight about the component, but please keep in mind those lists are non-exhaustive and only the main TPMs are described.

#### Web application

The website is made using [React](https://react.dev/). We decided to use this framework because it is the most popular, widely used in industry and greatly documented. Furthermore, the whole team was already familiar with React thanks to several past projects.

As for the component library, we chose [Chakra UI](https://chakra-ui.com/). Even though we didn't have any experience with it, we decided to try it out since it was visually appealing and popular. It is also very well documented and supported by the community.

[TypeScript](https://www.typescriptlang.org/) was a must-have for us to ensure code quality and to avoid bugs. Using a strongly typed codebase is also a good way to ensure the maintainability of our project in the future.

The website sends requests to the client API with the use of Axios HTTP Client. Axios has several advantages as compared to the more well-known Fetch API. It [supports more browsers](https://github.com/axios/axios#browser-support), has built-in protection against CSRF (Cross-Site Request Forgery) and converts data to the JSON format automatically. 
All of this makes Axios one of the most straightforward solutions to use.
To simplify the development of the connection with the API, our team has used the [OpenAPI Generator](https://openapi-generator.tech/) tool which generated TypeScript objects and functions for every controller and endpoint in our API. The auto-generated code uses Axios to send requests, and is based on the OpenAPI specification of the API which was conveniently provided to us by NestJS once we added the `SwaggerModule`.

**Website structure**

The web application offers to users several public and protected routes, which give access to the main PaaSTech features. 

An anonymous user can view:

- The home page
- The login and sign-up pages
- The email verification and password reset pages

On top of that, an authenticated user has access to their personal dashboard, which includes:

- their profile
- their projects
- their projects' information
- actions and logs for a given project
- environment variables
- project settings (which just allow to delete a project for now)

See the schema of the app routes below to better understand how the web interface is structured:

```mermaid
flowchart LR
    A{App} --> B[Public]
    A --> C[Protected]
    B -->|'root' /| D(HomePage)
    C -->|/dashboard/| E(DashboardHomePage)
    D -->|login| F(LoginPage)
    D -->|register| G(RegisterPage)
    D -->|email-verification/:token| H(EmailVerificationPage)
    D -->|password-recovery| I(PasswordRecoveryPage)
    D -->|password-reset/:token| J(PasswordResetPage)
    E -->|profile| K(ProfilePage)
    E -->|:projectId| L(DashboardDetails)
    L -->|logs| M(LogsTab)
    L -->|env| N(EnvironmentTab)
    L -->|settings| O(SettingsTab)
```
As mentioned earlier, the website is divided into two parts: publicly-accessible and protected. Public endpoints are those related to user authentication, including password recovery and account activation via email validation. The `:token` parameter used both for email verification and password reset is a UUID value which a user receives by email. This URL parameter is mandatory as it is used by the API to find a corresponding user in the database and either validate their account or authorize a password reset (cf. [Database Architecture](#database-architecture)).

The routes which allow users to access their profile and projects are not available for anonymous visitors. When accessing the `/dashboard` route, a user can view a list of all their existing projects. Clicking on one of the list items will open a page with more details about a particular project. The `:projectId` parameter in the page URL is thus the UUID of a project in the database, it is sent to the API to fetch additional data about the project.

#### CLI (Command-Line Interface)

The CLI tool, or Command Line Interface tool, is the main way for a User to create and deploy Projects. It has been developed in [Go](https://go.dev/) for its ease of use, its cross-platform compatibility and its overall great performances. Moreover, the Go language is well suited for CLI development, as it is a compiled language. It is used in many other CLI tools, such as Docker, Kubernetes, Terraform, and more.

In order to make this tool, the [Cobra library](https://cobra.dev/) was used. It allows for easy creation of CLI tools, with namely subcommands and flags. It is notably used by the Kubernetes CLI tool, `kubectl`, or even by companies like [Fly.io](https://fly.io/) and [Scaleway](https://www.scaleway.com/) for their own CLI tools.

Apart from Cobra, other libraries are used in this tool, such as [`go-git`](https://github.com/go-git/go-git), which allows for easy Git repository management. This is required for the tool to be able to initialize projects by adding a remote Git repository, and to deploy projects by pushing the code to our git server.

#### Client API

The Client API is one of the fundamental parts of our project. It not only handles all Client interactions, but is also the only application that can write any data in the shared database. It notifies [Pomegranate](#client-applications-deployment), the deployment manager, each time a project needs to be deployed and lets a user add their SSH keys to be able to connect to the local Git server. Every user request is sent and verified by the API before being carried out.

With this API being such an important part of the project for the Client, the choice of language and frameworks was very important to allow for maximal performance. It is made using [NestJS](https://nestjs.com/) with [TypeScript](https://www.typescriptlang.org/).
Being a cutting-edge framework gaining more and more attention, NestJS is very versatile and stands
out from the competition by proposing two things:
1. A [list of support modules](https://www.npmjs.com/search?q=%40nestjs), which totals to 41 as of today;
2. An [extensive documentation with code examples](https://docs.nestjs.com/) ranging from basics to difficult recipes to setup.

NestJS' support modules provide a substantial support for the integration of well-known modules into the application. The main TPMs used for the API are:

- `class-validator`: validation of DTOs (Data Transfer Objects) and their properties before they are handed out to NestJS controllers
- `@nestjs/config`: ease of access to a config service, centralising both environment variables and `.env` files
- `passport` and `passport-jwt` (along with `@nestjs/passport` and `@nestjs/jwt`): layer of abstraction to handle different authentication methods, such as JWTs, OAuth2 or OIDC. Here, only the JWTs are used.
- `uuid`: generation of UUIDs v4, following [RFC 4122](https://www.rfc-editor.org/rfc/rfc4122)
- `nodemailer` with `handlebars`: sending of outbound emails through SMTP, with templating of HTML emails sent
- `@nestjs/swagger` with CLI plugin: generation of OpenAPI specification (used by the web interface to generate API-related code) and running Swagger UI to simplify testing

Moreover, all of the support that NestJS provides doesn't make it slower. Benchmarks open-sourced in v7.6.13 ([available here](https://github.com/nestjs/nest/blob/e7fa96022e8b8580413490101683aabe387ca9b9/benchmarks/all_output.txt)) gave the following comparison:

- NestJS-express is nearly on par with Express for the average response time (`65.44 ms` versus `61.88 ms`)
- NestJS-express is twice as fast in maximum response time (only `325 ms` against Express' `747 ms`)
- NestJS-express can handle nearly twice as many requests concurrently, for the minimum amount doable (`14183 req/s` versus `8407 req/s`)
- NestJS-express is on par with Express for the number of requests handled concurrently, for the average amount doable (`15640` versus `16454.41`)

The API is currently running on NestJS-express v10.0.3, which has just been released. Two years have passed since the last benchmarks, and we can expect NestJS to be even faster as of now.

#### Git: controller & server

The Git Controller was made in Rust for its high performance, reliability, robustness and memory safety using the [tonic](https://github.com/hyperium/tonic) framework, which is a Rust implementation of gRPC. It was chosen because it is the most used gRPC framework in Rust, and is well documented.

We made the choice to use gRPC instead of a REST API. Indeed, since it uses [Protocol Buffers](https://developers.google.com/protocol-buffers), it is easier to maintain and to evolve as the application grows. It is also more performant than a REST API, and is easier to use, since it is strongly typed. It also limits communication problems between teams and services, since the communication is well-defined and documented through the Protocol Buffer file.

The Git Server is written in Go, for its simplicity and its performance. It is also well documented, and has a lot of libraries to help us build the server, unlike Rust which was the original choice. The libraries used are [ssh made by Gliderlabs](https://github.com/gliderlabs/ssh) which is used to create git clients and servers. It is also used by [Gitea](https://about.gitea.com) to create their git server.

#### Client applications deployment

Pomegranate, the application and deployment manager, is in charge of monitoring and managing the containers' lifecycle, as well as configuring them.

The [Rust](https://www.rust-lang.org/) programming language is used to develop the service.
We used it because we wanted an efficient and reliable language that could be trusted.
Thanks to its compiler, we are driven to write better code that handles errors and memory safety is guaranteed.

Furthermore, it supports the other technologies we use through the crates ecosystem.
Crates are third-party packages that provides additional features. They are managed by Cargo, which handles dependency resolution and versioning.

Establishing communication with other services is done via [gRPC](https://grpc.io/docs/what-is-grpc/core-concepts/).
gRPC imposed itself as the go-to framework for internal APIs thanks to its [high performance](https://www.nexthink.com/blog/comparing-grpc-performance) compared to [HTTP performance](https://github.com/programatik29/rust-web-benchmarks/blob/master/result/hello-world.md#comparisons).
Synthetic benchmarks shows that:
 - gRPC processes ~25% more requests compared to HTTP on a single core
 - gRPC scales better horizontally

To handle containerized client applications, we chose [Docker](https://www.docker.com/) because we already
knew how to use it, and it is straightforward to communicate with the Docker server to manage containers.
It additionally provides us with easy networking and storage APIs, so we don't have to worry about those issues.
This choice helped us to quickly set up the MVP (Minimum Viable Product) and get a working product as soon as possible.

Communication to the Docker socket was achieved through the [Bollard](https://crates.io/crates/bollard/) Rust crate.
Bollard is quite a powerful crate that allows us to do everything we need for this iteration:

- Starting and stopping containers
- Managing images
- Fetching logs and resource usage

Once containers are spawned, they are exposed via [Traefik](https://traefik.io/traefik/).
It is used because it discovers when containers are started/stopped and can dynamically reconfigure itself to create and delete routes as those events happen.

#### Database

The requirement for an ACID (Atomicity, Consistency, Isolation, and Durability) database led us to choose a relational database.
For our application there wasn't a need to have a multitude of adaptable fields and the benefits of having consistent and structured data with relationships between different objects were more important factors.
Moreover, the need for transactions is materialized in the fact that we are in a modular architecture. Transactions are the only way to ensure secure and complete actions between the different components.

The only flexible field is the environment variables that are stored for each Project, but since our chosen database, [PostgreSQL](https://www.postgresql.org/), supports the [JSONB](https://www.postgresql.org/docs/9.5/datatype-json.html) type, this was easy to implement.

As a market leader, PostgreSQL stands out for its performance and widespread use throughout the world. Not only is it free and open-source, it also offers a larger set of extensions and features than other databases, like the support for more than 43 different [data types](https://www.postgresql.org/docs/current/datatype.html). All of this together with its flexibility, the usage of [Multi-Version concurrency Control (MVCC)](https://www.postgresql.org/docs/current/glossary.html#GLOSSARY-CONCURRENCY) and our prior experiences made it a more interesting option that its major competitor [MySQL](https://www.mysql.com/).

The API connects to the database using an ORM called [Prisma](https://www.prisma.io/). Prisma also automates the generation and migration of the database schema as well as creating the classes associated to each table. Being an ORM that is both performant and allows the reorganisation of SQL data into simple classes, it was an important addition to our technology stack. Its complete documentation, active development and strong support made it more interesting than its competitors like [TypeORM](https://typeorm.io/) or [Objection JS](https://vincit.github.io/objection.js/).

#### CI/CD

Each repository has a GitHub Actions CI to execute format check, linter and tests at each push. This ensures a good code quality and a good test coverage for our entire project.

Most projects use our own Actions available in a [central repository](https://github.com/paastech-cloud/.github). 
This ensures that all projects have the same CI/CD pipeline and that the CI/CD pipeline is easily maintainable.
Moreover, thanks to Rust easy documentation creation process, a simple CI job is able to build the documentation and deploy it on [GitHub Pages](https://pages.github.com/) for every Rust project.
Furthermore, at every tag push on main, or on a workflow call, a Docker image is built for every project that has this workflow set up.
This image is then pushed as a repository package to the [GitHub Container Registry](https://ghrc.io) and is available for use.

On top of that, the web UI is automatically deployed on [Github Pages](https://pages.github.com/).

#### Deployment

To deploy every application of PaaSTech on a single machine, a `docker-compose.yml` was made alongside a `setup.sh` script to set up the host machine easily and deploy the entire project.
As of today, PaaSTech is not scalable and is not meant to be deployed on multiple machines.

Furthermore, even if the setup script does a bit of configuration, some folders still have to be created manually, the `.env` file needs to be filled and a [custom script](https://github.com/paastech-cloud/git-repo-manager/blob/main/hooks/post-receive) must be added in a specific directory on the host machine (`/srv/hooks` by default). 
This deployment method is available in [PaaSTech' `infra` repository](https://github.com/paastech-cloud/infra).

In this `docker-compose.yml`, we create 2 networks, one for the internal applications, and one for the client applications.
Each application is then deployed in a container, with environment variables that can be set up by the user in the `.env` file, and volumes as needed.
This file also ensures that:
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
- An Application (also referred to as Deployment) is an atomic unit of code and is the result of a Project deployment. This unit is internally managed and can only be configured to a certain extent by the Client.

### Component interaction

```mermaid
flowchart LR
    cli["CLI"]

    web <-- HTTP --> api
    cli <-- HTTP --> api
    cli <-- Git over SSH --> ssh


    subgraph Github Pages
        web["Web Frontend"]
    end

    subgraph PaaSTech internal services
        api["API"]
        git["Git controller"]
        ssh["OpenSSH server"]
        pom["Pomegranate"]
        db[(PostgreSQL)]
        fs[("`Host filesystem`")]
        docker(["Docker"])
        cr(["`Local container registry`"])

        api -. gRPC .-> git
        api -. gRPC .-> pom
        api -. SQL .-> db

        git -. manages repositories .-> fs
        ssh -. git receive-pack .-> fs
        ssh -. push images .-> cr

        pom -. manages deployments .-> docker

        docker -. use images .-> cr

        cr === fs
    end
```

In this schema, we can see the different components interacting with each other.

For this first iteration, we decided to split services to allow for a modular architecture. Doing so allows us to easily replace a component if needed, and to scale each component independently.

- The client API is the main component that is used by the clients to interact with the service. This API communicates with the Git controller for repositories management, with Pomegranate to create Docker deployments and with the database to store the data.
- The Git controller interacts with the host file system to create repositories, and with the local container registry to push images.
- Pomegranate interacts with the local container registry to pull images and with Docker to handle deployments.

Most of the interactions between the different components are done through gRPC, except for the web frontend and the CLI, which use HTTP requests. Using gRPC allows for a more efficient communication between the components.
All communications between internal services are done through a private network.


To understand their interactions better, we can look at an example of a project deployment by a client.

```mermaid
sequenceDiagram
    actor c as Client
    participant api as API
    participant db as Database
    participant git as Git Controller
    participant ssh as SSH Server
    participant pom as Pomegranate
    actor fs as File system
    actor docker as Docker

    note over c: Create a project
    rect rgba(181, 107, 110, 0.7)
        c ->>+ api: create project
        api ->>+ db: create project
        db ->>- api: OK
        api ->>+ git: Send GRPC request
        git -) fs: Create project
        git ->>- api: OK
        api ->>- c: OK
    end

    note over c: Add SSH key
    rect rgba(139, 164, 193, 0.7)
        c ->>+ api: Create SSH key
        api ->>+ db: Create new SSH key
        db ->>- api: OK
        api ->>- c: OK
    end

    note over c: Push project to remote
    rect rgba(167, 129, 171, 0.7)
        c ->>+ ssh: Push project
        ssh -) fs: Store project
        ssh -) docker: Build & save image
        ssh ->>- c: OK
    end

    note over c: Deploy a project
    rect rgba(115, 184, 170, 0.7)
        c ->>+ api: Deploy project
        api ->>+ pom: Deploy project
        pom -) docker: Deploy with built image
        pom ->>- api: OK
        api ->>- c: OK
    end
```

As described previously, the API manages most of the user interactions and redirects them to the right service. The only time the Client interacts directly with other applications, is to push their project to the server.

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

As you may notice, every table has a UUID field as a primary key. Compared to [Serial](https://www.postgresql.org/docs/current/datatype-numeric.html) that is often used to identify a table, a UUID guarantees a better uniqueness across the whole database. 
To generate a [UUID](https://www.postgresql.org/docs/current/datatype-uuid.html), we are using the UUIDv4 standard, which generates each UUID randomly. Thus, there are roughly 103 trillion UUIDv4s, lowering the chance of finding a duplicate UUID to one-in-a-billion. 
In comparison, Serials are limited to roughly 2 billion (2147483647) and will eventually clog up the column, effectively blocking the insertion of any new rows.
Even though serials take up less space (4 bytes) than a UUID (16 bytes) , incrementing integers offers information about the order of creation. This makes the table prone to timestamp-guessing. UUIDs are generated randomly and impossible to guess, completely removing this attack pattern.


Furthermore, since deployment URLs are built from the project id (`{projectId}.user-app.paastech.cloud`) , hiding the internal workings of the database was a requirement. 


The table `users` contains all the necessary information about each Client. Upon User creation, an `email_nonce` is automatically created. A User is only considered active once the email has been confirmed, signified by the field `email_nonce` being set to null.
A nonce is a random value that is unique and serves for identification. 
Should the User wish to reset their password, a new UUID will be saved in the field `password_nonce`. The Client will need to provide said nonce together with the new password to finish the procedure.
In the case of multiple reset requests, the field `password_nonce` gets overwritten each time. This ensures only the last request stays valid.
Once the password has been reset, `password_nonce` will be set to null.

The `ssh_keys` specified by each User allow them to push their repository onto our git server. Each SSH key can have a name to make it easier to distinguish multiple keys, however, it is not required.
An SSH key belongs to a Client and not a Project. Thus, the Client can access all of their repositories via any of their keys.


The `projects` table describes a Project. Its `config` field contains all the environment variables of said Project, like a database URL. 
Since the configuration changes for every Project, we decided to store it as a flexible JSON field. We decided to use a JSONB field that stores the JSON data in binary form, allowing for better performance than a simple JSON field.

### Detailed specification

#### Client API

##### Client states

When signing up, the client should provide an email address, a username and a password. They can then log into the software as soon as they verify their email. If needed, they can request a password reset using the "forgotten password" process. The account should be either deleted or locked/deactivated after it was active without a verified email for some time, although this is not implemented right now.

```mermaid
stateDiagram-v2
    c : Created
    v : Verified
    d : Deleted
    r : Reset mode

    [*] --> c
    d --> [*]

    c --> v: email verification

    c --> d: after some time

    v --> r: password reset request
    r --> v: password change

    v --> d: account deletion request

```

##### Authentication strategies

Regarding the authentication strategy, our team decided to create a `GET /auth/login` endpoint which returns both a JWT HttpOnly cookie and a Bearer token.
The cookies keep users safe from XSS (Cross-Site Scripting) attacks and are used for authentication by the PaaSTech web interface, while the Bearer token allows users to log in via the CLI.

##### Mail

Since sending and receiving emails is an important part of the user authentication and the password reset process, we needed a way to test these functions locally without needing to connect the application to a private email server every time.
After some search we came across [MailHog](https://github.com/mailhog/MailHog). MailHog allows anyone to create a temporary local SMTP server to send and receive emails through.
Even though you are not able to send emails to real email addresses, you can send and receive emails locally, which is very helpful for code testing.

On the other hand, once PaaSTech is deployed and accessible to anyone, it was connected to an SMTP email server.

##### GRPC

To communicate with other services, especially Pomegranate and the git-repo-manager, we used [gRPC](https://grpc.io/), due to its high performance and low latency. It also allows us to easily create NestJS gRPC clients in order to communicate with the other services. Another great advantage of gRPC is that it is language agnostic, which means that we can use it to communicate with services written in other languages. Creating the `.proto` files allowed us to define contracts between the services which made it easier to develop the services independently.

##### SSH key

To be able to push their projects to the Git server, each user needs to associate at least one SSH key to their account.
Using the command line, they will then be able to push their repositories' code to the server.

The API is responsible for the handling, storage and processing of the SSH keys. It acts as a "serving hatch" to store the keys in the database.
The git ssh server will then query the database directly to use compare the SSH keys sent by the Client to the ones in the database.

##### Administrators

In addition to the permissions of normal users, administrators have a few extra perks. 
Administrators are able to 
- view the non-critical information of each user (email, username)
- delete a user
- view all SSH keys
- view all existing projects

Since the SSH keys stored on our server are only the public part and therefore don't pose a security risk, there is no need to hide such information from administrators.
To avoid polluting the output if the administrator only wants to see their own SSH keys, we decided to separate both requests.
At the moment, it is neither possible to become an administrator through the website, nor to appoint someone to this role.

##### User Input Validation

One of the most important parts of an API is to check the Client input values. Every piece of data provided by the User should follow the asked type and rules to assure the best performance and security and minimize the risk of errors. To avoid checking each input individually, the verification is made using [Decorators](https://docs.nestjs.com/microservices/basics#decorators) and [Validators](https://docs.nestjs.com/pipes#class-validator) provided by NestJS. 
This way, the type and other necessary rules are automatically checked before even executing the code of the application and should the verification fail, the API will return a `BAD REQUEST` error.

##### Route protection

To protect each route from attacks, we secured each endpoint with [guards](https://docs.nestjs.com/guards). These classes define the rules to access different endpoints. To allow for the best user experience while keeping our application protected, three types of routes have been established:

- Public - accessible to everyone
- Private - only accessible to connected Clients
- Admin Only - only accessible to administrators

Each time a Client connects to a protected endpoint, the API guards automatically check if they have the necessary authorization before taking the actual request. 

##### Uniformity of response

In order to ease the communication with the other services, the API needed to return a uniform response. By using [Interceptors](https://docs.nestjs.com/interceptors), we were able to filter all outgoing data before it was sent to the Client. Every endpoint will return a json object containing a status as well as a message that contains the actual data to return.

#### CLI and login process

In order to log in from the CLI tool, the Client must have first created an account on the web frontend since the CLI tool does not allow for account creation.
This is intentional, and it ensures that creating an account cannot be automated. Using a website can also allow for more verifications, like [CAPTCHAs](https://en.wikipedia.org/wiki/CAPTCHA).

The login process begins with the user typing the `paastech login` command which would prompt them to enter their email address and password. The CLI tool then sends a request to the API to get a JWT token.
This token is stored locally in the user's config directory and is used for all subsequent commands requiring to be authenticated.
The token is valid for 6 hours, after which the user must re-authenticate against the API by typing the command again.

The token is stored in a configuration file following the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html).
This means the file is located under the `$HOME/.config/` directory on all systems supporting the [freedesktop.org](https://www.freedesktop.org/wiki/) recommendations (most likely GNU/Linux distributions).

This process is illustrated in the following sequence diagram:

```mermaid
sequenceDiagram
    actor u as User
    participant cli as CLI
    participant api as API
    participant db as PostgreSQL

    u ->>+ cli: paastech login
    cli ->>+ api: HTTP POST /auth/login
    api ->>+ db: SQL query
    db -->>- api: results
    api -->>- cli: JWT token
    cli ->>- u: Logged in successfully
```

#### Projects storage

As all PaaS providers, PaaSTech needs to store the code of the projects created by its Clients. This is done using a [Git](https://git-scm.com/) repository, which is created upon Project creation. The repository is then exposed to the Client, who can push their code to it.

We chose a push-based approach, instead of a pull-based one, because it is more adapted to the use case of PaaSTech. Indeed, the Clients are the only ones who can push to the repositories, and they are the only ones who need to be notified of changes. This is not the case for pull-based approaches, where the server needs to be notified of changes, and needs to pull the code from the repository. They also do not get the response as easily as with a push-based approach.

Besides, it forces the user to have a repository on a remote server, which is not the case for push-based approaches. And if they do, it might force them to adapt their current workstyle to the way our pull-based approach might work.

From our point of view, it might also be tedious to manage the access to the user's repositories.

We chose to use Git to store the code of the projects, because it is the most used version control system, and is the most adapted to store code. Using it makes the integration seamless for users who are probably already using Git.

##### Creating and deleting repositories

When a Client initializes a new Project, a new Git repository has to be created; it also has to be deleted when the Project itself is deleted.

The repository is created using the [`git init`](https://git-scm.com/docs/git-init) command. The repository is then stored in a directory, on the host machine, which is named after the **Project's id**.

For now, it is not scalable, as the git repositories are stored on the host machine. In future versions, the storage could be moved to a dedicated storage server or a distributed storage system.

In that sense, to prepare for future versions, the git controller is a **separated component** from the API which could have done the same and is also designed to be **stateless**. It does not store any information about the repositories and only uses the **Project's id** to name the repository's directory. This way, the git controller can be scaled horizontally, and, if needed, the repositories can be stored on a dedicated storage server after adding a layer of authorization.

Since the git controller is separated from the other components, it needs a way to communicate with the API. This was achieved using [gRPC](https://grpc.io/), which is a high performance RPC framework. It is used to create an interface other services can consume. The git controller can then receive commands from the rest of the services and execute shell commands to delete directories and git commands to initialize repositories.

Here's an overview of the interactions of the git controller:

1. Creation of a repository

```mermaid
sequenceDiagram
    participant API
    participant GitController
    participant HostMachine

    API ->> GitController: CreateProject(project_name)
    GitController ->> HostMachine: git init project_name
    HostMachine -->> GitController: Ok
    GitController -->> API: Ok
```

2. Deletion of a repository

```mermaid
sequenceDiagram
    participant API
    participant GitController
    participant HostMachine

    API ->> GitController: DeleteProject(project_name)
    GitController ->> HostMachine: rm -rf project_name
    HostMachine -->> GitController: Ok
    GitController -->> API: Ok
```

##### Authentication

Once the repositories are stored, the clients need to be able to push to them.

In order to let the users push to the repositories they created, we need to give them access to the host machine. An easy solution would be to give them an SSH access to the host machine with servers like [sshd](https://www.openssh.com/). However, this would give them access to the whole host machine, which is not what we want. Indeed, we want to limit the access of the clients to their own repositories, and not to the whole host machine.

One solution would be to use a git server like [Gitea](https://gitea.io/en-us/) or [GitLab](https://about.gitlab.com/), which provide a full-fledged git service, with authentication and authorization. However, this would require to maintain a full git server, with possibly duplicated data, which is not the goal of PaaSTech.

The option we chose is to create a custom git server, which handles the authentication and authorization of the clients, and then execute only the command `git-receive-pack` which is used to push to a repository. This way, the clients can only push to their own repositories, and not to the whole host machine.

The advantage of this solution is that it is lightweight and gives us full control over the authentication and authorization of the clients.
It also allows us to use the same authentication and authorization system as the rest of the Service, which is easier to maintain.

The authentication and authorization process is the following:

1. The client connects to the git server using SSH via `git push`
2. The git server checks if the client is authenticated using the SSH key provided by the client by matching it against the SSH keys stored in the database.
   1. If the client is not authenticated, the connection is closed
3. The git server checks if the client is authorized to push to the repository by checking if the client is the owner of the repository.
   1. If the client is not authorized, the connection is closed
4. The git server executes the command `git-receive-pack` which is used to push to a repository.

```mermaid
sequenceDiagram
    participant GitServer
    participant Database
    participant HostMachine

    Client->>GitServer: git-send-pack repository_name (git push)
    GitServer->>Database: Is the SSH key in the database?
    Database-->>GitServer: result
    alt Authenticated
        GitServer->>Database: Is the SSH key linked to the repository ?
        Database-->>GitServer: result
        alt Authorized
            GitServer->>HostMachine: Execute git-receive-pack
            HostMachine-->>GitServer: Push Successful (from host)
            GitServer-->>Client: Push Successful (continue)
        else Not Authorized
            GitServer-->>Client: Connection Closed (Not Authorized)
        end
    else Not Authenticated
    end
```

##### Building images

Once the code is finally pushed to the repository, it needs to be built in order to be deployed. Since the strategy we have chosen is [Docker](https://www.docker.com/), the code needs to be built into a docker image.

In that essence, since the client code can be written in any language, we need to be able to build the code in any language. To do so, we use [buildpacks](https://buildpacks.io/), which is a tool to build code in any language into a docker image. It is used by [Heroku](https://www.heroku.com/) to build their applications.

How can we trigger the build of the image? For a first version, we chose to use [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to trigger a bash script which will build the image on the host machine.

This is not scalable, as the build is done on the host machine.

It also poses questions about security risks since our SSH server needs to be able to access the docker daemon on the host machine. But it is a good first step to test the concept.

Some builders might not be able to build the code without configuration, hence, we need to be able to configure the buildpacks. To do so, the client can store a file called `buildpacks.yaml` in the `paastech` directory which will allow the user to configure the buildpacks. The file is a yaml file, which contains the configuration of the buildpacks. It is then used by the git hook to configure buildpacks. Here's the structure of the file:

```yaml
buildpacks:
  path: "."
  builder: "paketobuildpacks/builder:full"
  buildpack: "paketo-buildpacks/web-servers"
  env:
    - "BP_WEB_SERVER=nginx"
    - "BP_WEB_SERVER_ROOT=dist"
```

- `path` is the path to the directory containing the code to build
- `builder` is the builder to use to build the code
- `buildpack` is the buildpack to use to build the code
- `env` is the environment variables to pass to the buildpack

The buildpacks are then configured using the `pack` cli, which is the cli of buildpacks. The script is made in bash for its simplicity and its portability. It is also well documented, and has a lot of libraries to help us build the script.


#### Client Applications

Pomegranate is responsible for starting, stopping, and interacting with the client applications.
It is also responsible for managing the networking of the applications.

Pomegranate is completely stateless, and the API is the only external way to interact with it.
Once a container is started, Pomegranate does not keep track of it, and it is the responsibility of the execution engine to manage it.

```mermaid
flowchart TD
    web([port 80])
    webscure([port 443])
    traefik[Traefik]
    app_1[User app 1]
    app_2[User app 2]
    app_3[User app 3]

    rules{Resolve\nvia labels}

    webscure --> |HTTPS| traefik
    web --> |HTTP| traefik
    traefik --> rules

    rules --> |**uuid_1**.user-app:80| app_1
    rules --> |**uuid_2**.user-app:80| app_2
    rules --> |**uuid_3**.user-app:80| app_3

```

To expose newly spawned containers to the World Wide Web, [Traefik](https://doc.traefik.io/traefik/) is configured to listen to the 80 and 443 ports.
Then a set of labels is attributed to each container to create a unique subdomain `<app_uuid>.user-app.<fqdn>` redirecting to the port 80 of the associated app.

Pomegranate receives a domain name as an environment variable. 
Doing so enables flexibility during the development and deployment process, as the developer can easily swap between a localhost environment (`<uuid>.user-app.localhost`) and a production environment (`<uuid>.user-app.paastech.cloud`).

TLS termination is handled by Traefik. It manages resolving the DNS-01 challenge with the configured certificate authority. This allows us to have a valid certificate for all subdomains of PaaSTech.
In this case, our DNS registrar is Porkbun, and our certificate authority is Let's Encrypt.

## Post-mortem

### Organisational overview

In the beginning, we held a few meetings inviting everyone to specify all of the necessary functionalities. As a result of the large number of participants, we weren't really able to be as productive as we wanted but managed to create a first baseline of a specifications chart.

Afterwards, the work was split up into 3 teams of roughly equal size:
 
- The `client/web api` team had the goal to create both frontends (a CLI and a small web ui) as well as building the REST API backend.
- The `git controller` team was responsible for handling client code through git and building [OCI images](https://opencontainers.org) out of it.
- The `infrastructure` team which was tasked with running images and exposing the client application to the user.

All teams were originally formed using a preferential voting system, before getting slightly revised once we realized the real amount of work in each team. 
This really improved our productivity, but nevertheless, we ran into multiple problems of miscommunication or lack of communication inside as well as between the different teams, slowing down our progress. After some time the communication got better and we managed to handle the links between each service. However, there is still progress to be made regarding this issue.

### API

This project introduced most members of the team to NestJS. Due to its well written documentation it was easily picked up and with the various modules it allows for good performance and automized a lot during development.
The team could then apply best practices, like guards and interceptors, in order to develop more efficiently and with an overall better code quality.

However, we had some problems concerning the API as a whole. In the beginning, the API was meant to only manage the users.
A centralized controller should have managed the communication between the API, Pomegranate and the git server.
However, as more and more time passed, the controller was completely erased and the API dealt with most of its responsibilities. This led to an important increase of the workload, especially since it already had some delay in the early stages of the development due to miscommunication.

### Infrastructure

**Lessons learned:**

- Rust might not have been the best language to interact with Docker since Docker/Kubernetes has a first-party API in Go. We had a lot of Rust specific issues during development that slowed down how productive we were.
- We defined poorly how our service should be interacted with, forcing us to adapt interfaces at the last minute.
- Docker is far from ideal to build a PaaS on. A more reliable and secure option would be to use Kubernetes, ideally with microVMs.
