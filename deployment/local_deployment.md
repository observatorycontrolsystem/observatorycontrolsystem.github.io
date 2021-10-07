---
layout: page
title: Local Deployment
---

# Local or Network Deployment

This page will give some instructions on how to run the OCS applications in a production environment on one or more servers connected through a local network. If you are interested in a persistent, high availability cloud deployment procedure, check out the [Kubernetes Deployment]({% link deployment/k8s_deployment.md %}) section.

## Database Setup

Most of the OCS applications depend on a postgreSQL database. The Science Archive additionally requires the postGIS plugin to be installed in its postgreSQL database. It is possible to use the same postgreSQL instance across all the applications for a simpler setup, they will just each need their own database created. Your options for running postgreSQL are:

* Use an existing postgreSQL instance on your local network
* Install postgreSQL locally on your machine
* Run a postgreSQL docker container with postGIS installed

To start a postgreSQL instance with postGIS on a machine with docker forwarding its port 5432 for local usage, you can use:

```bash
docker run --name ocs-postgis -p 0.0.0.0:5432:5432 -e POSTGRES_PASSWORD=postgres -d postgis/postgis
```

{% include notification.html message="By default, postgreSQL within a docker container will store its data in the docker container's filesystem. If you want to persist the database, should should either create a docker volume or volume mount the postgreSQL data directory in the container to a folder on your local filesystem when you start the container. Check out the [docker docs](https://docs.docker.com/storage/volumes/) for more information." %}

After you have a postgreSQL instance running that you can connect to with a username and password, create a database for each OCS application you will use. These instructions assume the username and password are both set to the value `postgres`:

```bash
createdb --host 127.0.0.1 -p5432 -Upostgres -W my_db_name
```

{% include notification.html message="When running in production, it is recommended that database backups are created periodically. How frequently you create backups will depend on your needs. For example, creating one backup every day, and saving the last seven backups that were created would ensure that you would lose at most one day's worth of data, and that you could revert to any snapshot of the database taken within the last seven days. For information on how to create database backups, you can refer to the postgreSQL documentation [here](https://www.postgresql.org/docs/9.1/backup.html)."
status="is-info"
icon="fas fa-exclamation-triangle" %}

## Docker-Compose Templates

The preferred way to run each OCS application is using Docker. This way, you do not need to worry about installing the proper dependencies, or debugging strange interactions on your specific system. Template docker-compose.yaml files are provided within each OCS application repo to help getting setup to run the application in production. The docker-compose files should be setup in the repo to work and connect with one another considering a purely local deploy of all applications. If you are installing the OCS applications on different networked servers, you will need to update their environment variables to point to the correct location for each of their connections.

The preferred way to run each OCS application is using Docker. This way, you do not need to worry about installing the proper dependencies, or debugging strange interactions on your specific system. Each OCS application provides a Docker image that is available on [Docker Hub](https://hub.docker.com/u/observatorycontrolsystem), or you can build the Docker images yourself using the Dockerfiles in the OCS application repositories.

One way to configure and run one or more Docker containers is by using Docker Compose. A set of services that are run via Docker Compose are specified using a Docker Compose file. A single Docker Compose file can technically be used to start up all the different applications of the OCS, but we would not recommend doing that for production deploys. One issue with starting up all components together is that if any component needs to be upgraded, then all of the components would have to be restarted. Another issue is that that all applications would need to run on a single machine. Instead we recommend logically splitting up all the applications into separate Docker Compose files. Template docker-compose.yml files are provided within each OCS application repository to help you get started.

The Docker Compose files in each repository are set up to work and connect with one another given a purely local deploy of all applications. If you are installing the OCS applications on different networked servers, you will need to update their environment variables to point to the correct location for each of their connections. The Database name, user/password, and host/port should also be updated to the proper values for your setup:

```bash
DB_NAME=my_db_name
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=127.0.0.1
DB_PORT=5432
```

When starting the different Docker Compose stacks, note that the Observation Portal requires a running ConfigDB to start up, the Adaptive Scheduler and Downtime Database requires both the Observation Portal and ConfigDB to be running, and the Science Archive requires the Observation Portal to be running if you are using the user accounts in the Observation Portal for user authentication in the Science Archive.

If you are new to Docker and Docker Compose, or if you would like to know more, check out the following resources:

* [Information on how to install Docker](https://docs.docker.com/get-docker/)
* [Highly recommended tutorial to get you started using Docker and get you familiar with it](https://docs.docker.com/get-started/)
* [Brief overview of Docker Compose](https://docs.docker.com/compose/)


## Application Initial Setup

The OCS applications are modular, so you can pick and choose which pieces you need in your setup. At a very minimum though, you will likely want to run the Observation Portal for its User accounts and authentication, and the Configuration Database to define the properties of your telescopes and instruments. You will want to start by creating at least one superuser account on the Observation Portal. This can be done by running the Observation Portal's custom `create_user` Django management command from within the Observation Portal container, updating the username and password as needed:

```bash
python manage.py create_user --superuser --user test_user --password test_pass --token my-api-token
```

You can then also run a management command to create the ConfigDB credentials within the Observation Portal that will allow it to authenticate:

```bash
python manage.py create_application --user test_user --name ConfigDB --client-id configdb_application_client_id --client-secret configdb_application_client_secret --redirect-uris http://127.0.0.1:7000/
```

And then in the Configuration Database docker-compose.yaml, update the environment variables to use your credentials as created above.

```bash
OAUTH_CLIENT_ID=configdb_application_client_id
OAUTH_CLIENT_SECRET=configdb_application_client_secret
```

{% include notification.html message="Note if you are running other applications that connect to the Observation Portal for authentication, such as the Science Archive or Downtime Database, you will need to run the create_application management command once for each of those applications as well to set up their credentials. They should each have a unique client ID and client secret"
status="is-info"
icon="fas fa-exclamation-triangle" %}

You should then be ready to start the Configuration Database container and have access to its admin interface through your Observation Portal account credentials. See the [ConfigDB Setup]({% link deployment/configdb_setup.md %}) section for information on how and in what order to create your telescope representation in the Configuration Database.
