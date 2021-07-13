---
layout: page
title: Downtime Database
subtitle: Django application that manages periods of downtime on telescopes and instrument types
---

# Downtime Database

This Django application manages a REST API and web frontend for creating and retrieving periods of downtime on a telescope or instrument type. Downtime periods are time ranges in which scheduling is blocked in advance for any number of reasons, including maintenance, non-robotic usage, or certain instruments being offline.

This application is optional in the OCS ecosystem. It is useful to block out downtimes in advance so that the scheduler can schedule observations around those periods of downtime, which can help prevent interrupts to ongoing observations. If that is not a big concern, then simply switching the telescopes to inactive in configdb or changing the state of instruments to be unschedulable will prevent those resources from being scheduled on the same way a downtime block would.

A Downtime block requires the time range, and the site, enclosure, and telescope code it applies to. It can optionally accept an instrument type, if you would like the downtime to only apply to instruments of the choosen type on that telescope. It can also optionally accept a human-readable downtime reason, which is strictly used to inform the user in various web interfaces.
