---
layout: page
title: Local Deployment
---

# Local or Network Deployment

This page will give some instructions on how to run the OCS applications in a production environment on one or more servers connected through a local network. If you are interested in a persistant, high availability cloud deployment procedure, check out the [Cloud Deployment]({% link deployment/cloud_deployment.md %}) section.

## Database Setup

Most of the OCS application depend on a postgresQL database (with the postGIS plugin installed for the Science Archive). You can share the same postgres instance with all the applications, they will just each need their own database created, but they could share credentials for simplicity. Your options for running postgres are:

* Use an existing postgres instance on your local network
* Install postgres locally on your machine
* Run a postgres/postgis docker container

To start a postgresQL instance with postgis on a machine with docker forwarding it's port 5432 for local usage, you can use:

```bash
docker run --name ocs-postgis -p 0.0.0.0:5432:5432 -e POSTGRES_PASSWORD=postgres -d postgis/postgis
```

{% include notification.html message="By default, postgresQL within a docker container will store it's data in the docker containers filesystem. If you want to persist the database, should should either create a docker volume or volume mount the postgres data directory in the container to a folder on your local filesystem when you start the container." %}

After you have a postgres instance running that you can connect to with a username/password (Assuming postgres/postgres here, change as needed), create a db for each OCS application you will use:

```bash
createdb --host 0.0.0.0 -p5432 -Upostgres -W my_db_name
```

The docker-compose templates within each OCS application should then be modified to set the proper environment variables to point to your database. Note you should change the DB name, user/password, and host/port to the proper values for your setup:

```bash
DB_NAME=my_db_name
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=0.0.0.0
DB_PORT=5432
```

## Docker-Compose Templates

The preferred way to run each OCS application is using Docker. This way, you do not need to worry about installing the proper dependencies, or debugging strange interactions on your specific system. Template docker-compose.yaml files are provided within each OCS application repo to help getting setup to run the application in production. The docker-compose files should be setup in the repo to work and connect with one another considering a purely local deploy of all applications. If you are installing the OCS applications on different networked servers, you will need to update their environment variables to point to the correct location for each of their connections.

## Application Initial Setup

The OCS applications are modular, so you can pick and choose which pieces you need in your setup. At a very minimum though, you will likely want to run the Observation Portal for its User accounts and authentication, and the Configuration Database to define the properties of your telescopes and instruments. The Configuration Database depends on the Observation Portal for account authentication, so you will want to start up the Observation Portal first and create at least one superuser account. This can be done by running the django management command from within the Observation Portal container:

```bash
python manage.py create_user --superuser --user test_user --password test_pass --token my-api-token
```

You can then also run a management command to create the ConfigDB credentials within the Observation Portal that will allow it to authenticate:

```bash
python manage.py create_application --user test_user --name ConfigDB --client-id configdb_application_client_id --client-secret configdb_application_client_secret --redirect-uris http://0.0.0.0:7000/
```

And then in the Configuration Database docker-compose.yaml, update the environment variables to use your credentials as created above.

```bash
OAUTH_CLIENT_ID=configdb_application_client_id
OAUTH_CLIENT_SECRET=configdb_application_client_secret
```

{% include notification.html message="Note if you are running other applications that connect to the Observation Portal for authentication, such as the Science Archive or Downtime Database, you will need to run the create_application management command once for each of those applications as well to set up their credentials"
status="is-info"
icon="fas fa-exclamation-triangle" %}

You should then be ready to start the Configuration Database container and have access to it's admin interface through your Observation Portal account credentials. See the [ConfigDB Setup]({% link deployment/configdb_setup.md %}) section for information on how and in what order to create your telescope representation in the Configuration Database.
