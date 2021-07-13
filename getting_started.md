---
layout: page
title: Getting Started
---

# Getting Started

The Observatory Control System (OCS) provides an interface that allows astronomers to submit and manage observation requests and science data, allows observatories to manage observing schedules, and it also provides an interface to allow software at a telescope to update the status of an observation and to add data products to a science archive once the data has been collected.

There are many different ways to run the projects that are part of the OCS. The simplest way to get up and running and to start evaluating it for your own purposes is the run the [OCS example project](https://github.com/observatorycontrolsystem/ocs_example).

The OCS example project contains a Docker Compose file which starts up an [Observation Portal]({% link components/observation_portal.md %}), [ConfigDB]({% link components/configuration_database.md %}), [DowntimeDB]({% link components/downtime_database.md %}), and [Science Archive]({% link components/science_archive.md %}), which all provide web APIs to programmatically interact with them. It also starts up an [Adaptive Scheduler]({% link components/adaptive_scheduler.md %}), which runs continuously to create observing schedules from all the schedulable observing requests in the system. A simple web frontend for the Observation Portal is also provided. The OCS example project ties all these components together so that you get a functioning OCS stack.

The example project is configured to pre-populate its database with example data on startup. These example data include test administrator user accounts that can be used to interact with and update the state of the system, a ConfigDB resource that is configured to be available for observing, downtime slots in the Downtime Database to illustrate how to block out periods of scheduled downtime on a telescope or instrument, and also an observing semester, proposal, and a few example observation requests that the Adaptive Scheduler can turn into an observing schedule. See [here](https://github.com/observatorycontrolsystem/ocs_example#example-data) for more information on how this example data is generated.

Prerequisites to run the OCS example project:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/)

To start the example project:
```bash
git clone git@github.com:observatorycontrolsystem/ocs_example.git
cd ocs_example
git submodule init
git submodule update
docker-compose up
```

See here in the the [OCS example README](https://github.com/observatorycontrolsystem/ocs_example/blob/main/README.md#running-the-example) for details of how to access the different components that are started. You can also take a look at the [Docker Compose](https://github.com/observatorycontrolsystem/ocs_example/blob/main/docker-compose.yml) file to see how the different components in the OCS are tied together.

Note that while the example project starts up all the components of the OCS, if you find that you only want to use a subset of the OCS, you can do so. For example, if you have an existing archive for science data that you want to keep on using, you can leave the OCS Science Archive out of the stack. Similarly, if you have a different scheduling scheme in mind, you can write your own scheduler by coming up with some rules for how to turn observation requests into an observing schedule and then you can use the Observation Portal API to get observing requests, compute a schedule from those requests, and save the observing schedule back into the system again using the Observation Portal API.

Note that the OCS example project is only meant for testing, and that if you want to restart the stack, the entire stack should be removed and recreated.

To stop the example project and remove the Docker containers that were created:

```bash
docker-compose down
```

What can you do with the OCS example stack?
- Submit observation requests using either the Observation Portal API or the web frontend
- Watch as the Adaptive Scheduler turns observation requests into scheduled observations
- View all scheduled observations and observation requests, using either the Observation Portal API or the web frontend
- Update observatory resources in ConfigDB to specify resources available for scheduling and watch as the scheduler shuffles the observing schedule to only place observations on schedulable resources
- Submit periods of scheduled downtime to DowntimeDB and watch as the scheduler shuffles around the observing schedule, not placing any observation requests during periods of scheduled downtime

Once you've gotten a feel for the OCS, here are some ways you can expand the capability of this stack, or customize it for your own needs:
- Add ElasticSearch to your stack in which you can store data that the Adaptive Scheduler will use to keep track of whether a telescope is available for scheduling or not, for reasons such as bad weather or instrument failure.
- Ultimately you can test out integrating the OCS into your system by simulating updating observation statuses using the Observation Portal API.

## Next steps

To learn more about OCS capabilities and how to use it, you can take a look through the [Observation Portal]({% link components/observation_portal.md %}) observing request language documentation to understand how you can use the language to describe your observatory's observing capabilities. You can also take a look at the observatory configuration in the [ConfigDB]({% link components/configuration_database.md %}) to understand how you can model the resources in your own observatory. Beyond that, browsing the [Components]({% link components/index.md %}) section will give you a better idea of the purpose of all the components of the OCS and how they work together, taking a look through different [Topics of Interest]({% link topics/index.md %}) sections might help answer some more specific questions, and taking a look through the [API documentation]({% link api/index.md %}) of the different projects will show you how to use the OCS APIs.

Once you are at the point where you would like to run these projects in production, or would perhaps like to run these projects locally but in such a way where your data persists across restarts, you can take a look at the [Deployment]({% link topics/deployment.md %}) section.
