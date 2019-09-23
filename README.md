# Demo

Shows how to run jenkins and set it up in an incremental manner.

Running [docker-compose.yml](docker-compose.yml) that contains the following services:
* ldap - a simple openldap server that is prepopluated on startup using [bootstrap.ldif](ldap/bootstrap/custom.ldif)
* jenkins - My Bloody jenkins that watches changes from config.yml


## Prerequisites
* docker for mac/windows
* docker-compose

> Jenkins slaves needs access to the host IP. We use [host.docker.internal](https://docs.docker.com/docker-for-mac/networking/#known-limitations-use-cases-and-workarounds) for docker for mac and windows. If you are using docker on Linux, you will need to replace this alias with your actual host IP Address

> If you have a corparate proxy, please fill the proxy information inside [.env](step-by-step/.env) file.

### Starting up a clean jenkins

```shell
docker-compose up -d; sleep 30
open http://localhost:8080
```

### Adding LDAP Config
We will add an LDAP Authentication/Authorization

```shell
cat config-templates/01-ldap.yml >> config.yml; sleep 10
open http://localhost:8080
```

|username|password|groups|
---|---|--|
|bob.dylan|password|developers, team-leaders
|james.dean|password|developers|
|jenkins.admin|password|jenkins-admins
|jenkins.swarm|password|

### Adding credentials
We will add a simple user/password credential

```shell
cat config-templates/02-credentials.yml >> config.yml; sleep 10
open http://localhost:8080/credentials/
```

### Adding tools
We will add a Maven 3.5.0 auto installer

```shell
cat config-templates/03-tools.yml >> config.yml; sleep 10
open http://localhost:8080/configureTools/
```

### Adding docker cloud
We will add a docker cloud with 2 templates for different slaves:
* generic - uses [odavid/jenkins-jnlp-slave:latest](https://github.com/odavid/jenkins-jnlp-slave)
* cloudbees-tools - uses [cloudbees/jnlp-slave-with-java-build-tools](https://github.com/cloudbees/jnlp-slave-with-java-build-tools-dockerfile)

```shell
cat config-templates/04-docker-cloud.yml >> config.yml; sleep 10
open http://localhost:8080/configure
```

### Adding dsl scripts
We will add a dsl script to generate 2 pipeline jobs for 2 applications:
* [apps/nodejs-calc](apps/nodejs-calc) - A simple nodejs calculator with mocha-tests
* [apps/java-calc](apps/java-calc) - A simple Java calculator with junit tests

Both jobs are running on a dynamic slave and pushing their test results to jenkins.

```shell
cat config-templates/05-dsl-scripts.yml >> config.yml; sleep 10
open http://localhost:8080/
```

Seems like chucknorris plugin missing let's run: 
```shell
docker-compose -f docker-compose-chucknorris.yml up -d
```

## Cleaning up
```shell
# Terminate the main project
docker-compose down --remove-orphans --volumes
docker-compose rm jenkins
```
