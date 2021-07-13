---
layout: page
title: Adaptive Scheduler
subtitle: Python script that manages scheduling requests into observations.
show_sidebar: false
menubar: menu
hide_hero: true
---

# Adaptive Scheduler

A python script that continuously runs the scheduling loop, optimizing the effective priority over the scheduling horizon. It utilizes a [Mixed Integer Programming (MIP) solver](https://www.gurobi.com/resource/mip-basics/) to optimize for the largest effective priority on the telescope network over your observing horizon. It uses [Googles OR Tools](https://github.com/google/or-tools) for the solver, which supports multiple free or commercial optimization algorithms. It has a large number of configurable options via command line or environment variable that are described in the [README](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/README.md).

## How it works

The scheduler gets data from the other OCS components to know what, when, and where it can schedule. A diagram showing the flow of a scheduling cycle and its communication with the other components is shown below.

![Scheduler Workflow](/assets/images/scheduler_workflow.png)

The [Configdb]({% link components/configuration_database.md %}) and [Observation Portal]({% link components/observation_portal.md %}) connections are essential, but the DowntimeDB and TelemetryDB connections are both optional. The scheduler starts by getting the set of schedulable telescopes, by looking at what sites and telescopes are `active` in ConfigDB and optionally by looking at an Elasticsearch DB for weather and telescope availability telemetry. It then gets the set of schedulable requests from the Observation Portal and checks what observations are currently running on each telescope. A set of pre-filters are applied to build the availability windows for each request on each telescope resource, and then the optimization kernel runs and solves for the optimal schedule. The output schedule of where and when to complete each scheduled request is then submitted to the Observation Portal as a set of observations on each telescope. Each scheduling cycle first schedules all `RAPID_RESPONSE` requests, and then schedules the `NORMAL` and `TIME_CRITICAL` requests together afterwards.

### Telemetry

The TelemetryDB currently expects weather and telescope availability telemetry in a specific format within an ElasticSearch database, but the code can be extended to support other data sources and formats for weather data. The format of data it currently expects is referenced in the [code here](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/monitoring/elasticearch_telemetry.py), while the a [NetworkStateMonitor](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/monitoring/monitors.py#L25) can be implemented for any other data source. The monitor will be queried each scheduling run and should be able to answer the question of if an observing resource (a telescope) is available for scheduling.

### Objective Function

The scheduling kernel optimizes to maximize the total effective priority of the scheduled observations on the network of telescopes. Effective priority is calculated differently based on the `observation_type`, as described in the table below.

| observation_type | effective priority |
| ---------------- | ------------------ |
| NORMAL | proposal priority * IPP value * request duration in minutes |
| TIME_CRITICAL | proposal priority * 100.0 * 60.0 (fixed one hour duration) |
| RAPID_RESPONSE | proposal priority |

The priority in the kernel is slightly weighted towards completing requests earlier in time, if all other factors are equal, and there is a slight perturbation applied to each requests effective priority for numerical stability of the solver.
