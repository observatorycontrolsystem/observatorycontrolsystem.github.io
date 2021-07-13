---
layout: page
title: Observation Portal
subtitle: Django Application that manages Users, Proposals, Requests, and Observations
---

# Observation Portal

The Observation Portal is a Django application that manages users, proposals, observation requests, and scheduled observations, and is the backbone of the Observatory Control System. The Observation Portal is deployed as a PyPI library called [django_ocs_observation_portal](https://pypi.org/project/django-ocs-observation-portal/), and contains Django apps for accounts, requestgroups, observations, proposals, and sciapplications.

## Django Apps

### Accounts

This app provides the user accounts that are used for authentication between all OCS applications, using [oauth2](https://django-oauth-toolkit.readthedocs.io/en/latest/). It also provides a **Profile** model which includes additional account details.

### RequestGroups

The OCS Request language is designed to be able to support a broad range of observation requests. The models in the diagram below show the hierchary of properties of a RequestGroup.
![Request Models](/assets/images/request_models.png)

- **RequestGroup** - The top level of the model. It contains one or more Requests and acts as a logical grouping to help the end user keep track of a set of Requests
- **Request** - A single independent set of observing parameters to be scheduled in one contiguous block on a resource
- **Location** - A set of restrictions on which resources the Request can be scheduled on
- **Window** - One or more datetime intervals during which the Request could be observed
- **Configuration** - A set of observations on a single target
- **Instrument Configuration** - A set of exposures using certain instrument settings
- **Guiding** - Parameters for the guiding of the observation
- **Acquisition** - Parameters for the acquisition of the target
- **Constraints** - A set of observing constraints that must be met to schedule the observation
- **Target** - The parameters describing the target, supporting ICRS, orbital elements, hour angle, or satellite coordinates

More information about the specific fields expected in each section of the RequestGroup can be found in the [API specification]({% link api/downtime.md %}). Most fields have their values validated automatically from the acceptable values defined in the [Configuration Database (Configdb)]({% link components/configuration_database.md %}). There are several areas of the configuration which allow for user defined instrument properties that will be validated automatically, including things like modes and optical path elements.

#### Observation Type

A RequestGroup can be submitted with one of three observation types, described in the table below. Each observation type has its own pool of allocated time to draw from, so users must be allocated time in the observation types they want to submit with.

| observation_type | description |
| ---------------- | ------------------ |
| NORMAL | a normal Request whose priority is just the proposal's base priority multiplied by the `ipp_value` |
| TIME_CRITICAL | base proposal priority is multiplied by 100. This factor is meant to beat out any NORMAL request, but all TIME_CRITICAL requests of the same priority are treated equally. This type is meant for observations that are coordinated between multiple observatories, or just have an extremely high priority |
| RAPID_RESPONSE | These requests bypass normal scheduling and have their own scheduling loop that can pre-empt any non-RAPID_RESPONSE observation that is currently in progress. All requests of this type with the same base proposal priority are treated equally. These observation_types are very disruptive to the schedule, so they should only be used for observations that truly must begin immediately |
| DIRECT | a request of this type is created when a directly submitted observation is created to store the details of that observing request. These should only be submitted outside of the normal scheduling range of time or set of instruments, or during a scheduled period of downtime, so that they don't interfere with the scheduling of other observations |

#### Extra Params

Most of the levels of a RequestGroup have a JSON field called `extra_params` which accepts arbitrary key/value pairs that will be passed along to the telescope control software. This allows for complex non-standard configuration, instrument, target, or guiding and acquisition parameters to be added and used without modifying the Observation Portal code. Data passed through the `extra_params` fields will not be validated by default except in a handful of special cases, but custom validation code can be added to overridden serializers.

### Observations

The observations app deals with scheduled or directly submitted observations. An Observation is a scheduled request, or a request plus information about status, where the observation is scheduled, and when. The hierarchy of an Observation is shown below, but it is very similar to that of the Request except with a few additional pieces of information tacked on.
![Observation Models](/assets/images/observation_models.png)

- **Observation** - The top level of a scheduled Request, this contains the *where* and *when* information,
as well as some of the RequestGroup fields that the telescope control software might want to include in the observation's data products
- **Configuration Status** - This structure contains status information about a single Configuration of the Request.
Observatory software is expected to report back status per Configuration in real time as it is attempted and completed.
The contents of this structure will be merged into its corresponding Configuration when retrieved through the API.

Observations can be scheduled directly through the `/api/schedule/` endpoint, which will create a corresponding RequestGroup of `observation_type` **DIRECT**. They can also be created referencing existing Requests using the `/api/observations/` endpoint, which will just create an Observation and Configuration Statuses referencing the existing Request along with a start and end time and a location defined by the site, enclosure, telescope, and instrument.

### Proposals

Proposals are an essential part of the Observation Portal for authenticating who is allowed to submit observation requests and on what resource. Proposals can have one or more Principle Investigators (PIs) and any number of Co-Investigators (COIs) assigned to them. They can have any number of Time Allocations associated with them for any number of observing Semesters. The book keeping for the different types of time allocated to each proposal is handled automatically within the Observation Portal as observation requests are submitted and cancelled and scheduled observations are completed and updated. Proposals also contain a numerical priority, which is used when scheduling with the [Adaptive Scheduler]({% link components/adaptive_scheduler.md %}) as part of the optimizations objective function.

Proposals are currently added to the system in one of two ways - either through the Django admin interface, or by generating them via the science application process in the sciapplications app.

### Sciapplications

This Django app handles the proposal submission and review process. Calls for science applications can be created via the admin interface, and eligible users can submit science applications via the API for consideration. After review, the accepted science applications can be automatically converted into proposals with time allocations for the coming observing semester. This application can be completely ignored by observatories with existing proposal submission and time allocation processes - they can instead opt to just input the final proposals into the system.

## How to Customize

The first level of customization comes from the instrument and observatory definitions within Configuration Database. By using generic modes, optical path elements, and custom validation_schema, a lot of unique validation rules can be added to the Observation Portal without modifying any code. For more advanced customization, the [Observation Portal Project](https://github.com/observatorycontrolsystem/observation-portal-project) provides a forkable project that uses the `django_ocs_observation_portal` apps. It supports customization via overriding any serializer in the project, or overriding the `as_dict` methods of models which are used for generating API responses. Environmental variables defined in the [README](https://github.com/observatorycontrolsystem/observation-portal) are used to specify the dotpath of each overridden class or method.

### Serializer Overrides

The serializers define the validation logic for all the models of the Observation Portal. Any serializer can be overridden by defining a serializer class that subclasses the corresponding library serializer and overrides any serializer method - usually the `validate` method. Here is an example serializer override that enforces that spectrograph instruments must include calibrations with their spectrum.

```python
from rest_framework import serializers
from django.utils.translation import ugettext as _

from observation_portal.requestgroups.serializers import RequestSerializer
from observation_portal.common.configdb import configdb

SPECTRO_CATEGORY = 'SPECTRA'

class MyRequestSerializer(RequestSerializer):

    def validate(self, data):
        validated_data = super().validate(data)
        spectral_configuration_types = set()
        for configuration in validated_data['configurations']:
            instrument_type = validated_data['instrument_type']
            if configdb.get_instrument_type_category(instrument_type).upper() == SPECTRO_CATEGORY:
                spectral_configuration_types.add(configuration['type'])
        if not set(['SPECTRUM', 'LAMP_FLAT', 'ARC']).is_subset(spectral_configuration_types):
            raise serializers.ValidationError(_('SPECTRA instrument observations must include an ARC and LAMP_FLAT in addition to the SPECTRUM'))
        
        return validated_data
```

### as_dict Overrides

We use `as_dict` methods within the Django models to efficiently serialize return values for API calls. This circumvents the normal serialization behaviour within the [Django Rest Framework Serializers](https://www.django-rest-framework.org/api-guide/serializers/), but it greatly speeds up the call, which is important when you have a large number of observation requests. Overriding the as_dict method allows you to add fields or non-model data to any API response. The `as_dict` methods can be overridden in the same way as the serializer classes, by specifying a new dotpath to your custom method in an environment variable. A simple as_dict override example is below.

```python
from observation_portal.requestgroups.models import requestgroup_as_dict

def my_requestgroup_as_dict(instance):
    output = requestgroup_as_dict(instance)
    output['my_new_field'] = 'this is my new field'
    return output
```
