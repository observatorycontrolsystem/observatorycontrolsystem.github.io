---
layout: page
title: Components
show_sidebar: false
menubar: menu
hide_hero: true
---

# Components

The Observation Control System is composed of several python libraries, applications, and django web applications that work together to provide a complete upper stack solution to an observatory. The diagram below shows the major OCS components and how they are interconnected.

![OCS Components](/assets/images/ocs_applications.png)

## Core Components

The core of the OCS is the [Observation Portal]({{ site.baseurl }}{% link components/observation_portal.md %}). It manages User accounts for all the other applications that need authentication, and it provides a database and API for submitting, updating, and viewing Proposals, Requests, and Observations.

The [Configuration Database (Configdb)]({{ site.baseurl }}{% link components/configuration_database.md %}) manages the state of the observatory, as viewed by the other upper stack applications. It provides hierarchical models of the observatory's structure, from the site/enclosure/telescope down to the modes and properties of cameras within an instrument. The purpose of this information is to allow of automatic validation and filling in of observatory details in the other applications.

These two applications represent the minimum viable portion of the OCS needed to manage Users, Proposals, and Requests for an observatory.

## Optional Components

These applications all add on to the core OCS applications in various ways, but their inclusion into the software stack is optional. If an observatory already has a custom solution for one of the components, like scheduling or archiving of data, then those can be integrated with the Observation Portal rather than using the OCS [Adaptive Scheduler]({{ site.baseurl }}{% link components/adaptive_scheduler.md %}) or [Science Archive]({{ site.baseurl }}{% link components/science_archive.md %}).

The [Adaptive Scheduler]({{ site.baseurl }}{% link components/adaptive_scheduler.md %}) provides a python application which interfaces with the other OCS databases to turn a set of input Requests into a set of scheduled Observations in the Observation Portal. While the scheduler is fully integrated into the OCS ecosystem, it is possible to use another scheduling method instead.

The [Science Archive application and Ingester library]({{ site.baseurl }}{% link components/science_archive.md %}) provide the means to upload data products into Amazon Web Services (AWS) S3 buckets, and store their metadata in a database to allow for easy searching, filtering, and bulk downloading of the data products by users. AWS S3 provides a relatively cheap and scalable way for observatories to store their data products redundantly and long-term, but if an observatory already has a solution for storing data products, they can continue to use that.

The [Downtime Database]({{ site.baseurl }}{% link components/downtime_database.md %}) is a simple database, REST API, and web frontend for creating and retrieving periods of unschedulable time on a telescope or instrument_type. Using the downtime database isn't strictly necessary, but it can increase scheduling efficiency at the observatory since observations can be scheduled exactly around the bounds of known periods of downtime.
