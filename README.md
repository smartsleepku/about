# SmartSleep

The [SmartSleep](https://www.smartsleep.ku.dk/) project is a sleep study for the [Faculty of Health and Medical Sciences](https://sund.ku.dk/). The SmartSleep project is a research project about sleep.

These repositories contain the source code for all components involved in gathering the sleep data submitted by the research participants.

The project is composed of multiple parts:

 - Mobile applications
   - [iOS](https://github.com/smartsleep/ios) application
   - [Android](https://github.com/smartsleep/android) application
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

[Android Studio](https://developer.android.com/studio) was used for building the Android application.

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

# Notes on setting up development environment for Smartsleep

Install docker, docker-compose.

Clone repositories:
```
git clone git@github.com:smartsleepku/services.git
git clone git@github.com:smartsleepku/android.git
cd services
git submodule update --init --recursive
git submodule update --remote
```

Install prerequisites:
```
sudo apt install gradle
```

Add `services/auth/.env` file:
```
APP_ID=auth
PORT=3000
LOG_LEVEL=warn
REQUEST_LIMIT=100kb
SESSION_SECRET=eXeilu8AiPoQuae7aeth8jaiZ7phugh9Chain5mie0zaebiemeebei0iegoong3S # use some secret (not sure how that session secret is used)

#Swagger
SWAGGER_API_SPEC=/spec
```

Create the following file:
```
stack.env
```
You can use the following file as a template:
```
stack.env.sample
```

Create ssreports directory:
```
mkdir <base_dir>/ssreports
```
Create the following file:
```
report-builder/config.ini
```
You can use the following file as a template:
```
report-builder/config.ini.sample
```

Create the following file:
```
secrets/admin-config.ini
```
You can use the following file as a template:
```
secrets/admin-config.ini.sample
```

And load environment variables before running docker compose commands:
```
source stack.env
```

Now run `docker-compose up` from `services` directory to start backend servers.

When servers are running, you need to populate the databases with appropriate data so local client can communicate with local backend:
```
docker exec -it services_db_1 bash
mongo mongodb://root:oJuwu7Tohquaongoh9Nooz9vaThaeche@db:27017
use smartsleep
db.clients.insert({clientId: "android", clientSecret: "jiejov6ohn3iexiig5EifiLaiyai5eizoo4avaeQueesohqu5Iezohg7ooyohc1o", authorized: true})
db.clients.insert({clientId: "ios", clientSecret: "jiejov6ohn3iexiig5EifiLaiyai5eizoo4avaeQueesohqu5Iezohg7ooyohc1o", authorized: true})
db.clients.insert({clientId: "survey", clientSecret: "jiejov6ohn3iexiig5EifiLaiyai5eizoo4avaeQueesohqu5Iezohg7ooyohc1o", authorized: true})
db.users.insert({emailAddress: "test@example.test", password: "test", attendeeCode: "5"})
db.users.insert({emailAddress: "ios1@example.test", password: "ios1", attendeeCode: "ios1"})
db.users.insert({emailAddress: "android1@example.test", password: "android1", attendeeCode: "android1"})
db.users.insert({emailAddress: "android2@example.test", password: "android2", attendeeCode: "android2"})
exit
exit
```

`clientSecret` in the snippet above should correspond to the `CLIENT_SECRET` in the android and ios files explained below.

For the android and ios clients, do the following:
Add `android/app/src/main/java/dk/ku/sund/smartsleep/manager/ClientSecret.kt` file:
```
package dk.ku.sund.smartsleep.manager

import com.github.kittinunf.fuel.core.FuelManager

val CLIENT_SECRET = "jiejov6ohn3iexiig5EifiLaiyai5eizoo4avaeQueesohqu5Iezohg7ooyohc1o"
val ADMIN_CREDENTIALS = "Eiphuthe4saiPiu7imoweitaek5caey9ohhohN0uh9aegh5pot7Uer2aeToXie2u"

fun configure() {
    FuelManager.instance.basePath = "http://10.0.2.2:18080"
}
```

And for ios the following file:
```
import Foundation

// Admin password
let AdminCreds = "Eiphuthe4saiPiu7imoweitaek5caey9ohhohN0uh9aegh5pot7Uer2aeToXie2u"


// Production
let ClientSecret = "jiejov6ohn3iexiig5EifiLaiyai5eizoo4avaeQueesohqu5Iezohg7ooyohc1o"
let baseUrl = "http://localhost:18080"
```

Values for `CLIENT_SECRET` and `ADMIN_CREDENTIALS` in the file above could be anything for development, but it should equal to the corresponding values in the databases.

The url http://10.0.2.2:18080 is host system address for proxy service, as seen from android emulator in android studio.
Same goes for ios, but host IP address is just localhost.

Now you can use android studio to run emulator and test code.  After starting the app in emulator, enter the code `5`, and then e-mail `test@example.test`.

## Updates in code

When you update one of the submodules, or docker-compose file, run the following:
```
git submodule update --remote
docker-compose build
```

# Deploying to a staging server

## Prepare repositories

First, clone and checkout backend repositories on the server:
```
git clone git@github.com:smartsleepku/services.git
cd services
git submodule update --init --recursive
mkdir mongodata
```

### Configure activity
Create and edit the following file:
```
activity/secrets/database-production.properties
```
You can use the following file as a template:
```
activity/secrets/database-production.properties.sample
```

Make sure there are no more than 1 endline characters in the end of the .properties file.  Otherwise database config won't be loaded.

### Configure auth
Create and edit the following files:
```
auth/.env
auth/config/production.json
```
You can use the following files as templates:
```
auth/.env.sample
auth/config/production.json.sample
```

### Configure survey
Create the following file:
```
secrets/config-production.php
```
You can use the following file as a template:
```
secrets/config-production.php.sample
```

### Configure firebase
Download firebase admin sdk configuration from firebase dashboard:
```
secrets/firebase-adminsdk.json
```

## Configure environment
Create the following file:
```
stack.env
```
You can use the following file as a template:
```
stack.env.sample
```

## Configure report and admin servers

```
mkdir <base_dir>/ssreports
```
Create the following file:
```
report-builder/config.ini
```
You can use the following file as a template:
```
report-builder/config.ini.sample
```

And add docker secret from it:
```
sudo docker secret create db_config report-builder/config.ini
```

Create the following file:
```
secrets/admin-config.ini
```
You can use the following file as a template:
```
secrets/admin-config.ini.sample
```

And add docker secret from it:
```
sudo docker secret create admin_config secrets/admin-config.ini
```

## Set up server
Run the following commands from the services directory:
```
docker swarm init # initialize the computer as swarm control node
docker node ls # check that it worked
source stack.env
docker-compose -f docker-compose.staging-new.yml build # this rebuilds local docker images
docker stack deploy -c docker-compose.staging-new.yml smartsleep # this starts all services
docker service ls # see that services are running
```

You can use the following commands to check services:
```
docker service ps <service> # check particular service
docker logs <service> # check service logs (you can use autocompletion)
```

To stop services, run:
```
docker stack rm smartsleep
```

Now, when you started services for the first time, you need to fill database with keys to be able to test:
```
docker exec -it smartsleep_db.1.<press tab to autocomplete> bash
mongo mongodb://root:oJuwu7Tohquaongoh9Nooz9vaThaeche@db:27017
use smartsleep
db.clients.insert({clientId: "android", clientSecret: "<INSERT CLIENT SECRET FOR ANDROID>", authorized: true})
db.clients.insert({clientId: "ios", clientSecret: "<INSERT CLIENT SECRET FOR IOS>", authorized: true})
db.clients.insert({clientId: "survey", clientSecret: "<INSERT CLIENT SECRET FOR SURVEY>", authorized: true})
db.users.insert({emailAddress: "ios1@example.test", password: "ios1", attendeeCode: "ios1"})
db.users.insert({emailAddress: "ios2@example.test", password: "ios2", attendeeCode: "ios2"})
db.users.insert({emailAddress: "ios3@example.test", password: "ios3", attendeeCode: "ios3"})
db.users.insert({emailAddress: "android1@example.test", password: "android1", attendeeCode: "android1"})
db.users.insert({emailAddress: "android2@example.test", password: "android2", attendeeCode: "android2"})
exit
exit
```

To access the services from outside, we need to point incoming https traffic to staging server domain, for example, `smartsleep.example.test`, to proxy service running in docker on port 18080.  This can be done using nginx reverse proxy with the following settings:
```
upstream smartsleep {
  server 127.0.0.1:18080;
}
server {
  listen 80;
  server_name smartsleep.example.test;
  location / {
    return 301 https://$server_name$request_uri;
  }
}
server {
  server_name smartsleep.example.test;
  access_log /var/log/nginx/smartsleep.example.test-access.log;
  error_log /var/log/nginx/smartsleep.example.test-error.log;
  location / {
    proxy_pass         http://smartsleep;
    proxy_redirect     off;
    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
  }
  listen 443 ssl; # managed by Certbot
  ssl_certificate /etc/letsencrypt/live/example.test/fullchain.pem; # managed by Certbot
  ssl_certificate_key /etc/letsencrypt/live/example.test/privkey.pem; # managed by Certbot
  include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
  ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```
provided you have used letsencrypt to install https certificates.

## Test limesurvey access
If you want to test limesurvey access from desktop app, generate a token using the following:
```
docker exec -it smartsleep_auth.1.<press tab to autocomplete> bash
LOG_LEVEL=info npx ts-node bin/auth-client.ts --clientId survey --clientSecret <INSERT CLIENT SECRET FOR SURVEY>
exit
```
Copy the resulting token and paste it into desktop app script in `desktop/src/index.js` after `Bearer `.

Also, you should create mysql user with privileges for limesurvey:
```
docker exec -it smartsleep_sql.1.<autocomplete> bash
mysql -u root -p # and then ender root password from compose file
CREATE USER 'survey' IDENTIFIED BY 'Eingee7ooco7jaewoh5Suuy7sha3thae';
GRANT ALL PRIVILEGES ON *.* TO 'survey' IDENTIFIED BY 'Eingee7ooco7jaewoh5Suuy7sha3thae';
exit
exit
```

Now you can run `npm start` from `desktop` and follow the steps on screen to activate limesurvey:
* database location: sql
* database user: survey
* database password: <password for sql user>
* database name: smartsleep
* table prefix: lime_

## Test android app

Create the following file:
```
app/src/main/java/dk/ku/sund/smartsleep/manager/ClientSecret.kt
```
You can use the following template:
```
app/src/main/java/dk/ku/sund/smartsleep/manager/ClientSecret.kt.sample
```

Build .apk package, install it on android and use attendeeCode which you inserted into the database before.

## Test ios app

Create the following file:
```
SmartSleep/Services/ClientSecret.swift
```
You can use the following template:
```
SmartSleep/Services/ClientSecret.swift.sample
```

Also download from your firebase dashboard a .plist file and save it into:
```
SmartSleep/GoogleService-Info.plist
```

Install app on iphone and use attendeeCode which you inserted into the database before.

## Test mongodb access.

Make sure port 27017 is opened to incoming connections.  This is needed to access mongodb from desktop client.

Download client from https://docs.mongodb.com/compass/master/install/
Start it, connect to the server and browse the database.

# Production deployment

## Backups
### Backup mysql database
`mysqldump -u root -p -A > ./backup-<date>.sql` and type root password (supply corresponding date)

### Backup mongodb
Run the following (change `path/to/backups` to the corresponding path)
```
docker run -it --rm -v /path/to/backups:/path/to/backups mongo:xenial bash
mongodump --host smartsleepdb.sund.root.ku.dk --username admin --password <admin-password> --out path/to/backups/mongo-backup-<date>
exit
```

## Prepare repositories

```
git clone git@github.com:smartsleepku/services.git
cd services
git submodule update --init --recursive
```

### Configure activity
Create and edit the following file:
```
activity/secrets/database-production.properties
```
You can use the following file as a template:
```
activity/secrets/database-production.properties.sample
```

Make sure there are no more than 1 endline characters in the end of the .properties file.  Otherwise database config won't be loaded.

### Configure auth
Create and edit the following files:
```
auth/.env
auth/config/production.json
```
You can use the following files as templates:
```
auth/.env.sample
auth/config/production.json.sample
```

### Configure survey
Create the following file:
```
secrets/config-production.php
```
You can use the following file as a template:
```
secrets/config-production.php.sample
```

### Configure firebase
Download firebase admin sdk configuration from firebase dashboard:
```
secrets/firebase-adminsdk.json
```

## Configure report servers
```
mkdir <base_dir>/ssreports
```
Create the following file:
```
sudo docker secret create db_config report-builder/config.ini
```
You can use the following file as a template:
```
report-builder/config.ini.sample
```

## Set up server
Run the following commands from the services directory:
```
docker node ls # check that it worked
/usr/local/bin/docker-compose -f docker-compose.production.yml build # this rebuilds local docker images
```

If services are running, stop them before re-deploying:
```
docker stack rm smartsleep
```
and wait 10 seconds (to make sure docker removed old network).  Then deploy services:
```
docker stack deploy -c docker-compose.production.yml smartsleep # this starts all services
docker service ls # see that services are running
```

You can use the following commands to check services:
```
docker service ps <service> # check particular service
docker logs <service> # check service logs (you can use autocompletion)
```

To stop services, run:
```
docker stack rm smartsleep
```

## Re-init survey
Go to `desktop` repository.  If it's not configured yet, do the following:
```
docker exec -it smartsleep_auth.1.<press tab to autocomplete> bash
LOG_LEVEL=info npx ts-node bin/auth-client.ts --clientId survey --clientSecret <INSERT CLIENT SECRET FOR SURVEY>
exit
```
Copy the resulting token and paste it into desktop app script in `desktop/src/index.js` after `Bearer `.
Then start the application:
```
npm start
```
and use respective settings from `services/secrets/config-production.php` to re-init the survey.
