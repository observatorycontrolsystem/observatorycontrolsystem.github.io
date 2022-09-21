---
layout: page
title: Scheduler Custom Interfaces
subtitle: Integrating Custom Interfaces for Weather and Seeing to the Adaptive Scheduler
---

# Scheduler Interfaces
The Adaptive Scheduler has interfaces to retrieve weather / telemetry information and seeing information for telescopes. The built in implementations assumes that the data exists within an OpenSearch index with a specific format. In this guide, we will describe the format of the built in interfaces, and also provide details on how to write custom interfaces to retrieve the information elsewhere.

## Weather / Telemetry Interface
At the beginning of each scheduling run, the Adaptive Scheduler retrieves the current state of the network and constructs a list of what telescopes are available to schedule on that run. The only thing that the scheduler really cares about is whether a telescope is available or not at that moment in time - it doesn't really matter why a telescope is unavailable as the end result is the same - no scheduling will take place on that telescope. This is most important for networks with more than one telescope because it allows observations to be re-scheduled on an available resource when the original resource becomes unavailable.

### Built in Data format
The built in Data format relies upon using two pieces of information, whether a telescope is available or not, and if not, why it is not available. We call a piece of telemetry a *datum*, and the datum name of the required datums are **Available For Scheduling** and **Available For Scheduling Reason**. The first datum should have its value be a boolean that is *True* if telescope is available, *False* otherwise. The reason is a string reason that should be a concatentation of any reasons the Telescope Control System software wants to report back up the stack. This reason is purely used by the scheduler to be stored in the log on each scheduling run. The fields for each datum that are expected in the OpenSearch index defined by the environment variables **OPENSEARCH_INDEX** and **OPENSEARCH_URL** are listed in the table below:

| Field                  | Description                |
| ---------------------- | -------------------------- |
| datumname              | The name of the telemetry item |
| datuminstance          | The iteration of the telemetry item with this name (usually 1) |
| site                   | The configdb site code for this piece of data |
| observatory            | The configdb enclosure code for this piece of data |
| telescope              | The configdb telescope code for this piece of data |
| value_string           | The value of the telemetry item |
| @timestamp             | The timestamp at which this data was recorded in OpenSearch (recorded time) |
| timestamp              | The timestamp this piece of data first entered its current value (start time) |
| timestampmeasured      | The latest timestamp this value was recorded at (end time) |

It is expected that the current value of these datums are constantly reported into the OpenSearch index with updated **@timestamp** and **timestampmeasured** fields, even if the value hasn't changed. If the **@timestamp** is greater than 15 minutes old, the telescope will be marked as unavailable as it is assumed it is no longer accessible over the network.

### Custom Implementation
If you have an existing telemetry storage solution you would like to hook into, the telemetry interface code has been written with subclassing and customizing it in mind. The steps to support a custom interface are detailed below:

1. Fork the [Adaptive Scheduler project](https://github.com/observatorycontrolsystem/adaptive_scheduler)
2. Subclass the [NetworkStateMonitor](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/monitoring/monitors.py#L25). Implement the `retrieve_data` method to query your data source and return a list of dictionaries, each one reprensenting the availability of a single telescope. They should have a *telescope*, *observatory*, and *site* key to describe which telescope it applies to, and it should have some key describing if the telescope is available or not. Implement the `is_an_event` method which will be called on each item in the retrieved data list and return True or False if that telescope is available for scheduling or not. Finally, implement the `create_event` method to create an [Event](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/monitoring/monitors.py#L21), which just has a type, reason, and start and end time. Any telescope which has an event created is considered to not be available - the type and reason of the event just describe why and are recorded in the log for that run.
3. Modify the set of monitors used [here](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/monitoring/network_status.py#L73) to add in your new custom monitor implementation. You shouldn't need to remove the existing ones - the OpenSearch telemetry monitor will automatically not be used if no OpenSearch URL and index name are specified in the environment.

#### Example Telemetry Monitor implementation:

```python
class MyCustomNetworkStateMonitor(NetworkStateMonitor):
    def retrieve_data(self):
        # Hit whatever external service is needed to retrieve telemetry/weather data 
        # for your sites and put it into a format like this. The reason, start, and end
        # are optional and can be omitted if your service doesn't supply them.
        telemetry = [
            {
                'site': 'tst',
                'observatory': 'doma',
                'telescope': '1m0a',
                'available': False,
                'reason': 'Weather: humidity too high',
                'start': datetime(2022, 1, 1, 5, 4, 22),
                'end': datetime(2022, 1, 1, 5, 6, 44)
            },
            {
                'site': 'lsc',
                'observatory': 'doma',
                'telescope': '1m0a',
                'available': True,
                'reason': '',
                'start': datetime(2022, 1, 1, 5, 5, 33),
                'end': datetime(2022, 1, 1, 5, 6, 44)
            }
        ]
        return telemetry
    
    def is_an_event(self, datum):
        # This receives each element from the list returned in retrieve_data and
        # decides if it is an event or not, based on your data format
        return datum.get('available', False)

    def create_event(self, datum):
        # This creates an Event tuple out of a datum that is deemed to be an event
        # using the data in the datum to populate event fields
        event = Event(
            type='UNAVAILABLE',
            reason=datum.get('reason'),
            start_time=datum.get('start'),
            end_time=datum.get('end')
        )
        return event
```

## Seeing Data Interface
A source of current seeing data for each telescope on the network is required for the *max_seeing* constraint to work in requests. The current seeing value is pulled down from a data source at the beginning of each scheduling run, and then used to block off potential windows in which a request with a seeing constraint is visible at each telescope. The **SEEING_VALID_TIME_PERIOD** environment variable represents the amount of time in minutes to block out at a telescope following a seeing constraint that isn't met by the current seeing value. This defaults to 20 minutes, and should be set using the knowledge of how stable seeing values are at your telescope and how frequently they will be reported. It is ideal if you can get the current seeing value very frequently (on the order of a scheduling run), and then the time you block off could just be on the order of how long observations are on your network.

### Built in Data format
The built in Data format is basically the minimum set of information you need to describe seeing at a time on a telescope. It is expected to be stored in an OpenSearch index defined by the environment variables **OPENSEARCH_SEEING_INDEX** and **OPENSEARCH_URL**. The expected fields are described below:

| Field                  | Description                |
| ---------------------- | -------------------------- |
| site                   | The configdb site code for this piece of data |
| enclosure              | The configdb enclosure code for this piece of data |
| telescope              | The configdb telescope code for this piece of data |
| seeing                 | The value of the seeing in arcseconds at this telescope |
| @timestamp             | The timestamp at which this seeing value was recorded |

{% include notification.html message="Note: The seeing value will only be used as a constraint for **SEEING_VALID_TIME_PERIOD** minutes into the future from whatever the latest **@timestamp** is on that telescope" %}

### Custom Implementation
If you have an existing seeing storage solution you would like to hook into, the seeing interface code has been written with subclassing and customizing it in mind. The steps to support a custom interface are detailed below:

1. Fork the [Adaptive Scheduler project](https://github.com/observatorycontrolsystem/adaptive_scheduler)
2. Subclass the [SeeingMonitor](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/monitoring/seeing.py#L10). Implement the `update_data` method to query your data source and save a dictionary of resources to dictionaries containing the timestamp and seeing value. The resources as defined by ConfigDB are available in a list via the base class from `get_resources`, and each resource is a string concatentation of the fully qualified location of a telescope, which is `{telescope_code}.{enclosure_code}.{site_code}`. The seeing data should be saved within the class variable `self.current_seeing_by_resource`, and the method should return *True* if any seeing values have changed, or *False* if they are all the same since the last call to `update_data` was made. This is needed to support only re-running the scheduler if things have changed.
3. Modify the main scheduler entrypoint, [cli.py](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/cli.py) to import your new `MyCustomSeeingMonitor` and change the code [here](https://github.com/observatorycontrolsystem/adaptive_scheduler/blob/main/adaptive_scheduler/cli.py#L206) to instantiate your seeing monitor.

#### Example Seeing implementation:

```python
class MyCustomSeeingMonitor(SeeingMonitor):
    def update_data():
        previous_seeing_data = self.current_seeing_by_resource
        resources = self.get_resources()
        # Set your current seeing here from whatever data source you have
        # This is just an example of what the format should look like
        self.current_seeing_by_resource = {}
        for resource in resources:
            self.current_seeing_by_resource[resource] = {
                'seeing': 2.3,
                # This will only be enforced until 3:22:33 with a 20 minute validity time
                'time': datetime(2022, 1, 5, 3, 2, 33)
            }
        # return True if they are not the same, i.e. seeing values have changed
        return not previous_seeing_by_resource == self.current_seeing_by_resource
```
