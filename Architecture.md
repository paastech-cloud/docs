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

### Database architecture

***TODO: [CLIENT] describe the database architecture, as referenced in the MCD in the README***

### Detailed specification

#### Key constraints

***TODO: [CLIENT, INFRA] the key constraints that should never be broken by the application (or at least the external parts, like the API and container exposition) in order to maintain security, isolation and client data safety***

#### Client sign-up and login process

***TODO: [CLIENT] mermaid diagram and description of the login process flow, for both the CLI and the web frontend***

#### Projects storage

***TODO: [GIT] how are the projects stored and how is authentication handled upon push***

As all PaaS providers, PaaSTech needs to store the code of the projects created by its clients. This is done using a Git repository, which is created upon project creation. The repository is then exposed to the client, who can push its code to it.

We chose to use Git to store the code of the projects, because it is the most used version control system, and is the most adapted to store code. Using it makes the integration seamless for users who are probably already using git.

##### Creating and deleting repositories

When a user initializes a new project, a new repository has to be created, it also has to be deleted when the project is deleted.

The repository is created using the [git init](https://git-scm.com/docs/git-init) command. The repository is then stored in a directory, which is named after the **project's id**. It is stored on the host machine.

For now, it is not scalable, as the git repositories are stored on the host machine. In future version, the storage could be moved to a dedicated storage server, a distrubuted storage system or a bucket storage system.

In that sense, to prepare for future versions, the git controller is a separated component from the rest of the application, designed to be stateless, and only stores the repositories on the host machine. It does not store any information about the repositories, and only uses the project's id to name the repository's directory. This way, the git controller can be scaled horizontally, and, if needed, the repositories can be stored on a dedicated storage server after adding a layer of authorization.

Since it is separated it needs to communicate with the rest of the application. It was achieved using [gRPC](https://grpc.io/), which is a high performance RPC framework. It is used to create a bidirectional stream between the git controller and the rest of the application. The git controller can then receive commands from the rest of the application, and send back the result of the command.

We made the choice to use gRPC instead of a REST API. Indeed, since it uses [protobuf](https://developers.google.com/protocol-buffers), it is easier to maintain and to evolve as the application grows. It is also more performant than a REST API, and is easier to use, since it is strongly typed. It's support on different languages also makes it easier to use. It also limits communication problematics between teams and services, since the communication is well defined and documented through the protobuf file.

It was made in rust for its high performance, reliability, robustness and memory safety using the [tonic](https://github.com/hyperium/tonic) framework, which is a rust implementation of gRPC. It was chosen because it is the most used gRPC framework in rust, and is well documented.

##### Authentication

Once the repositories are stored, the clients need to be able to push to them.

In order to let the users push to thoses repositories, we need to give them access to the host machine. An easy solution would be to give them a ssh access to the host machine with servers like [sshd](https://www.openssh.com/). However, this would give them access to the whole host machine, which is not what we want. Indeed, we want to limit the access of the clients to their own repositories, and not to the whole host machine. And pose a great security risk.

One solution would be to use a git server like [gitea](https://gitea.io/en-us/) or [gitlab](https://about.gitlab.com/), which provide a full-fledged git service, with authentication and authorization. However, this would require to maintain a full git server, with possibly duplicated data, which is not the goal of PaaSTech.

The option we chose is to create a custom git server, which handles the authentication and authorization of the clients, and then execute only the command `git-receive-pack` which is used to push to a repository. This way, the clients can only push to their own repositories, and not to the whole host machine.

The advantage of this solution is that it is lightweight and give us full control over the authentication and authorization of the clients. It also allows us to use the same authentication and authorization system as the rest of the application, which is easier to maintain.

The git server is written in golang, for its simplicity and its performance. It is also well documented, and has a lot of libraries to help us build the server, unlike rust which was the original choice. The libraries used are [ssh made by gliderlabs](https://github.com/gliderlabs/ssh) which is used to create git clients and servers. It is also used by [gitea](https://about.gitea.com) to create their git server.

The authentication and authorization process is the following:

- The client connects to the git server using ssh
- The git server checks if the client is authenticated using the ssh key provided by the client by matching it against the ssh keys stored in the database
    - If the client is not authenticated, the connection is closed
- The git server checks if the client is authorized to push to the repository by checking if the client is the owner of the repository
    - If the client is not authorized, the connection is closed
- The git server executes the command `git-receive-pack` which is used to push to a repository

##### Building images

Once the code is finally pushed to the repository, it needs to be built in order to be deployed. Since the strategy we chose is [Docker](https://www.docker.com/), the code needs to be built into a docker image.

In that essence, since the client code can be written in any language, we need to be able to build the code in any language. To do so, we use [buildpacks](https://buildpacks.io/), which is a tool to build code in any language into a docker image. It is used by [heroku](https://www.heroku.com/) to build their applications.

How can we trigger the build of the image ? For a first version, we chose to use [git hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to trigger a bash script which will build the image on the host machine. This is not scalable, as the build is done on the host machine. It also poses questions about security risks since our ssh server needs to be able to access the docker daemon on the host machine. But it is a good first step to test the concept.

Some builders might not be able to build the code without configuration, hence, we need to be able to configure the buildpacks. To do so, the client can store a file called `buildpacks.json` in the `paastech` directory which will allow the user to configure the buildpacks. The file is a json file, which contains the configuration of the buildpacks. It is then used by the git hook to configure buildpacks. Here's the structure of the file:

```json
{
  path: "myapp",
  builder: "packetobuildpacks/builder:full",
  buildpack: "packeto-buildpacks/web-server",
  env: [
    {
        name: "PORT",
        value: "8080"
    }
    ...
  ],
}
```

- `path` is the path to the directory containing the code to build
- `builder` is the builder to use to build the code
- `buildpack` is the buildpack to use to build the code
- `env` is the environment variables to pass to the buildpack

#### Client Applications

***TODO: [INFRA] how are the Client applications provisioned, how they are exposed and how the underlying infrastructure is managed***

## Post-mortem

### Organisational overview

***TODO: [UNIFIED WORK] from an organisation standpoint, how was the entire team organised, how did the squads interact***
