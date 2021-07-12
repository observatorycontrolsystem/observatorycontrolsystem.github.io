---
layout: page
title: Observatory Control System
subtitle: Open source software for an API-driven observatory
show_sidebar: false
menubar: menu
hide_hero: false
# FIXME: If we go with this theme, fix this image url. For some reason when deployed on github pages it
# keeps resolving to the wrong url
hero_image: https://eheinrich.github.io/observatorycontrolsystem.github.io/assets/images/starry_sky.jpeg
---

With the field of astronomy undergoing a revolution in data volume and automation, many observatories
around the world are beginning to update their systems to take advantage of modern web technologies.
However, producing a fully-featured and maintainable Observatory Control System (OCS) is an expensive undertaking! Las
Cumbres Observatory successfully operates a network of 20+ robotic telescopes around the world, driven entirely
by APIs. The software that enables this has been bundled up and open-sourced, the goal of which is to increase
the rate of adoption of APIs in astronomical observing and to share the knowledge gained in the process of building
the software so that the entire community benefits.

**What does an API-driven Observatory Control System accomplish?**

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

See the [Getting Started]({{ site.baseurl }}{% link getting-started.md %}) guide for details on how to begin using the OCS.

# Questions?

You can reach out to us at [ocs@lco.global](mailto:ocs@lco.global).

{% include github.html %}

_The Open Source Observatory Control System Project is managed by Las Cumbres Observatory, with generous financial support from the Heising-Simons Foundation._

{% include logos.html %}
