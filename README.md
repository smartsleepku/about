# SmartSleep

The [SmartSleep](https://www.smartsleep.ku.dk/) project is a sleep study for the [Faculty of Health and Medical Sciences](https://sund.ku.dk/). The SmartSleep project is a research project about sleep.

These repositories contain the source code for all components involved in gathering the sleep data submitted by the research participants.

The project is composed of multiple parts:

 - Mobile applications
   - [iOS](https://github.com/smartsleep/ios) application
   - Android application (TBD)
 - Backend
   - [Authentication](https://github.com/smartsleep/auth) module
   - [Activity](https://github.com/smartsleep/activity) data service
   - [Notification](https://github.com/smartsleep/notify) service
   - [Survey](https://github.com/smartsleep/limesurvey) service
   - [Authenticating](https://github.com/smartsleep/proxy) proxy
 - [Desktop application](https://github.com/smartsleep/desktop) for survey configuration

All the backend components are referenced assembled in a [services](https://github.com/smartsleep/services) project which includes configurations and links to all the the services and docker-compose files for local development and for staging and production deployment.

## Architecture

The project is composed of mobile frontend applications, a backend which has multiple sub-components, and a backoffice desktop application for survey configuration.

Each individual component is described below.

### iOS Application

The iOS application is architected with Views, Controllers, and Services. The Views has view specific code which is needed for layout of components on screen. The controllers composite the data structures and handle user events. The Services handle data gathering and storage, from accelerometer data used in detecting sleep, storing data in local caches, and transmitting activity data to from the backend. The services also make composite night data from the activity data.

The views are created using Interface Builder, with the exception of special view components which could not be described using standard components, which are then found in the Views subfolder.

The controllers and services can be found in Controllers, and Services folders, respectively.

All user secrets (like credentials) are stored securely in the keychain encrypted storage on the device.

All cached data is stored in an embedded SQLite database in the application documents folder on the device.

### Backend

The backend is composed of multiple sub-components.

 - [Authentication](https://github.com/smartsleep/auth) module
 - [Activity](https://github.com/smartsleep/activity) data service
 - [Notification](https://github.com/smartsleep/notify) service
 - [Survey](https://github.com/smartsleep/limesurvey) service
 - [Authenticating](https://github.com/smartsleep/proxy) proxy
 
![Server layout](https://github.com/smartsleep/about/raw/master/smartsleep-server-layout.png "Server layout")

#### Authentication Proxy

The proxy is an [nginx](https://www.nginx.com/) webserver which authenticates all incoming requests before allowing access to either the activity microservice or the survey microservice. Authentication is done via [authentication subrequests](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-subrequest-authentication/) which sends a request to the authentication microservice which in turn evaluates the client and user credentials before allowing or denying access.

#### Databases

All user and activity data is stored in a [MongoDB](https://www.mongodb.com/) database with separate database users for accssing [PII](https://en.wikipedia.org/wiki/Personal_data) and SPI data. All data is stored with encryption-at-rest.

All survey data is stored in MySQL.

#### Authentication Microservice

Authentication is done via [JWT](https://jwt.io/) which are transmitted via bearer authentication headers in a TLS encrypted request to the backend via https. A request to a login endpoint on the authentication microservice with valid client id, client secret, and user credentials, will yield a JWT result which is then carried in all subsequent requests to the backend.

#### Activity Microservice

Whenever the user submits activity data, rest data is derived from it and stored alongside it for followup sleep data gathering.

#### Notification Microservice

Once the survey participants sleep rythms are submitted, the notification microservice will send background push notifications to mobile devices every morning prompting the device to upload all activity data in the background.

#### Survey Microservice

The survey microservice is a Limesurvey fork which is used to create surveys which survey participants are requested to partake in. The surveys responses are keyed by user id which in and of themselves contain no PII.

### Desktop Application

The backoffice desktop application is a portable electron application used to access the survey microservice. It carries a shared client secret which alongside a username and a password prevents anyone outside the study from accessing participant data.

## Building

### Prerequisites

Xcode is used for building the iOS application. [CocoaPods](https://cocoapods.org/) is used for dependency management.

[Docker](https://www.docker.com/get-started) and [docker-compose](https://docs.docker.com/compose/install/) is needed to build and deploy locally. [Docker-machine](https://docs.docker.com/machine/install-machine/) and Docker Stack is used to deploy to the staging server.

[Node 8](https://nodejs.org), and [OpenJDK 8](https://openjdk.java.net/install/) is used in development of the individual services. You should have Node 8 and a recent JDK installed locally. [nvm](https://github.com/creationix/nvm) is recommended for maintaining multiple versions of node locally.

[IntelliJ IDEA](https://www.jetbrains.com/idea/) has been used for development of microservices implemented in Kotlin and [Atom](https://atom.io/) has been used for microservices implemented in Typescript, but any IDE can be used for local development.

GPG is used for managing secrets.

### Submodules

Git submodules are used for maintaining dependencies. To pull all dependencies (if you did not perform a recursive clone) run:

```
git submodule update --init --recursive
```

### Decrypting secrets

Before starting the stack for the first time, secrets must be decrypted. To decrypt the secrets, run:

```
gpg --decrypt-files --yes `find . -path '*secrets*' -and -name '*.gpg'`
```


### Local deployment

To build and run the backend services locally, simply clone the services project including all submodules and run:

```
docker-compose up
```
