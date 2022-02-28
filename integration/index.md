---
layout: page
title: Custom Integration
---

# Custom Integration

This section describes the integration between existing or custom implementations of observatory software components.

## Telescope Control System (TCS)

A TCS provides the interface between the telescope subsystems and the Observation Portal. [This document]({% link integration/tcs.md %}) describes in detail the necessary interactions between the OCS Observation Portal and an observatory's TCS software.

Most observatories will have existing TCS software, so this document is written in order to present the capabilities that will need to be integrated into existing software so that it can communicate with the OCS Observation Portal.

## Custom Scheduler

[This document]({% link integration/scheduler.md %}) describes the role of the observatory scheduling software and presents a general overview of the role of a scheduler - to schedule observations on observatory resources based on existing requests from observatory users or staff.

The OCS offers an [Adaptive Scheduler]({% link components/adaptive_scheduler.md %}) which is designed to work directly with other components of the OCS, so this article is aimed at providing users who wish to implement their own scheduler the resources and information to do so.

## Observatory Data Flow

[This document]({% link integration/data_flow.md %}) describes the components necessary for an observatory to adopt the OCS Science Archive for data archival and retrieval. It covers data product ingestion after an observation, and contains some advanced topics with helpful information to help observatories construct a robust data transfer and archival system.
