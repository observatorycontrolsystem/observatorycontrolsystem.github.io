---
layout: page
title: Observatory Control System
subtitle: Open source software for an API-driven observatory
show_sidebar: false
menubar: example_menu
hide_hero: false
hero_image: {{ site.baseurl }}/assets/images/starry_sky.jpeg
---

With the field of astronomy undergoing a revolution in data volume and automation, many observatories
around the world are beginning to update their systems to take advantage of modern web technologies.
However, producing a fully-featured and maintainable Observatory Control System is an expensive undertaking!

A major strength of open source software is not having to reinvent the wheel.
Las Cumbres Observatory successfully operates a network of
20+ robotic telescopes around the world, driven entirely by APIs. Parts of the software that enable this are already
open source, with the rest of it to follow suit beginning mid-2020. The goal is to increase
the rate of adoption of APIs in astronomical observing, and to share the knowledge gained in the process of building the software so that the entire community benefits.

**So, what does an API-driven Observatory Control System accomplish?**

Astronomers can:

* Submit requests to observe a target, track the states of those requests, and cancel
requests if their needs have changed
* Be notified once their observation is complete
* Download their science data

Observatories can:

* Automatically update the observing schedule for a telescope when a new request is submitted
* Upload science data
* Update the states of requests to keep users in the loop and to keep track of what still needs to be done
* Manage users who are able to submit requests

**Since these actions are all backed by APIs, they can be triggered programmatically and can easily be incorported into other web interfaces and software systems.**

# Projects under the Observatory Control System

The Observatory Control System software provides for request and observation management, observation 
scheduling, and a science archive. An observatory that adopts this software has the option to use all 
of the parts, or only a subset of them. Specifically, the projects that make up the software are:

### Observation Portal

This Django application is the main interface that astronomers interact with to submit observation requests and to
monitor the status of those requests. It also stores the observing schedule that is generated from all observation 
requests by the scheduler. It is fully backed by APIs and includes modules for the following:

<dl>
  <dt>Proposal management</dt>
  <dd>Calls for proposals, proposal creation, and time allocation</dd>
  <dt>Request management</dt>
  <dd>Observation request validation, submission, and cancellation, and views providing
  ancillary information about them</dd>
  <dt>Observation management</dt>
  <dd>Store and provide the telescope schedule, update observations, and update
  observation requests on observation update</dd>
  <dt>User identity management</dt>
  <dd>Provides Oauth2 authenticated user management that can be
  used in other applications</dd>
</dl>

### Configuration Database

This Django application stores observatory configuration in a database and provides an
API to get that configuration, which is needed by the observation portal to perform automatic
validation and to calculate estimated request durations. It includes details on the
configuration of sites, enclosures, telescopes, instruments, and cameras. The camera configuration has
customizable sets of modes and optical path elements to support a wide range of
current and future instrument configurations. The configuration is also used by the scheduler to determine
available telescopes.

### Downtime Database

This Django application stores periods of scheduled telescope downtime in a database and provides an API
to retrieve those periods of downtime. Scheduled downtimes occur for a variety of reasons including maintenance and
education use. Downtimes are used in the validation of requests in the observation portal and are also
used by the scheduler to block out time that is not available.

### Scheduler

This Python application creates telescope observing schedules. It gets a set of observation requests
from the observation portal, computes a schedule, and then saves a set of scheduled observations
back to the observation portal. It currently uses the GUROBI solver to solve for the schedule but will provide
the option to use an open source solver instead.

### Rise-Set Library

This Python library wraps the FORTRAN library SLALIB. It performs visibility calculations
for requested targets in both the observation portal and the scheduler. It supports sidereal
and non-sidereal target types and includes airmass, moon distance, and zenith constraints
on visibility.

### Science Archive

This Django application provides an API to save and retrieve science data. Certain metadata are
stored in a database for easy querying and full image data are stored in AWS s3 for download.

### Ingester Library

This Python library aids in uploading data to the science archive.

# Project Conventions

All projects will follow clear conventions for consistency. Automated checks will be set up
to enforce these conventions by preventing code that is in violation from being incorporated
into the codebases.

### Code quality, guidelines, and style

The projects are all written in Python and should use at least version 3.6. Stylistically,
all projects should conform to the [PEP8](https://www.python.org/dev/peps/pep-0008/) standard.
New code will be required to pass a static code analysis check.

### 3rd party libraries

3rd party libraries will be updated regularly. Patches for security vulnerabilities will be
applied as soon as possible.

### Testing

The codebases of these projects all have test suites, which are vital for quickly checking that
the code works as intended. The projects all have varying degrees of test coverage. Each new
feature or fix should have a test confirming its functionality. An automated check will make sure
that test coverage does not decrease over time.

Besides running test suites, it is extremely useful for development to be able to
run a project locally on a development machine. Instructions will be provided for each project on
how to run them locally. In addition, basic helm charts and docker images will be provided for doing so.

### Releases

New versions of the projects will be released when there is new code available. Versioning
of projects will follow the guidelines of [semantic versioning](https://semver.org/). In addition, detailed
release notes will be provided on new features, fixes, or breaking changes.

### Documentation

General top-level documentation, which includes things like FAQs, descriptions for
how the different projects fit together, and usage examples will be made available in a central
location. In addition, documentation of public interfaces, which give a more precise definition of
how to use the libraries and web APIs, will be generated. Finally, each project will have a README, which
will describe what a project does, why it is useful, how to get started, and where to get more help if needed.

# Community

This is a project for the community, by the community!

### Code of Conduct

In order to foster a welcoming and inclusive open source community, all projects will be released with a
[Contributor Code of Conduct](https://www.contributor-covenant.org/version/2/0/code_of_conduct/). By
participating in the Observatory Control System community, you agree to abide by its terms.

### Support and Contributing

Please see the TOM Toolkit documentation for [Support](https://tom-toolkit.readthedocs.io/en/stable/support.html) and
[Contributing](https://tom-toolkit.readthedocs.io/en/stable/contributing.html). The TOM Toolkit has been an active open
source project since 2018, also developed and maintained by engineers at Las Cumbres Observatory. They have refined
their process, figuring out what works, so we want to put to good use what they have learned during that time by
following in their footsteps.

# Questions?

You can reach out to us at [ocs@lco.global](mailto:ocs@lco.global).

{% include github.html %}

_The Open Source Observatory Control System Project is managed by Las Cumbres Observatory, with generous financial support from the Heising-Simons Foundation._

{% include logos.html %}
