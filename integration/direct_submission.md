---
layout: page
title: Submitting Calibrations / Direct Submissions
subtitle: How to submit a Direct submission via the API
---

# DIRECT submissions
These special requests can be submitted into the Observation Portal to represent an observation that must take place at a fixed time and location, separate from the scheduler's scheduled observations. For these requests, the `observation_type` is `DIRECT`, denoting that they are a direct submission and excluding them from scheduling runs.

## When to use
Direct submissions can be used to schedule calibration frames that happen before or after the scheduleable night bounds (daytime calibrations), or to schedule during a night during a period of downtime when the scheduler is not scheduling. It is possible, but not recommended to mix DIRECT submissions during a night with scheduler scheduled observations - for night time calibrations it will work best to schedule them alongside all the science requests. 

{% include notification.html message="Note that Direct submission observations circumvent much of the normal validation of the request, so it is possible to submit observations which will be impossible to complete. Make sure the values you are submitting can be interpretted by your Telescope Control System (TCS)." %}

## How to submit
Direct submissions are essentially requests with added location, instrument, and date/time information specified - they will look exactly the same as scheduler scheduled observations, except the request's `observation_type` field will be `DIRECT`. The configurations must also have the specific `instrument_name` and `guide_camera_name` fields set, which are the instrument codes in ConfigDB from the instrument you are targeting. A direct submitted observation's request does not need the `window` or `location` blocks set, since this information is already present at the top level. There is no way in the supplied UI to submit direct observations, so it must be done via the api from your calibration scripts. Direct submitted observations can be POSTed to the `/api/schedule/` endpoint, with a payload of the form:
```json
{
    "site": "tst",
    "enclosure": "domc",
    "telescope": "2m0a",
    "start": "2019-08-24T14:15:22Z",
    "end": "2019-08-24T14:15:22Z",
    "request": 
    {
        "configurations": [{
            "constraints": {
                "max_airmass": 1.6,
                "min_lunar_distance": 0
            },
            "instrument_configs": [{
                "mode": "default",
                "exposure_time": 10,
                "exposure_count": 1,
                "optical_elements": {
                    "filter": "g"
                }
            }],
            "guiding_config": {
                "optional": true,
                "mode": "ON"
            },
            "instrument_type": "MyInstrumentType",
            "instrument_name": "fa01",
            "guide_camera_name": "es02",
            "type": "EXPOSE",
            "target": {
                "name": "MyFavoriteTarget",
                "type": "ICRS",
                "ra": 22.2,
                "dec": 33.3
            }
        }],
        "observation_note": "This is my favorite observation",
        "optimization_type": "TIME",
        "acceptability_threshold": 100,
        "configuration_repeats": 1,

    },
    "proposal": "MyProposal",
    "priority": 10,
    "name": "My requestgroup name"
}
```

