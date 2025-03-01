---
layout: page
title: ConfigDB Initial Setup
---

# Configuration Database Initial Setup

When you start up a Configuration Database for the first time, you will need fill in the initial representation of your observatory. This can either be done by logging into the Admin interface and filling in data structures in order, or by submitting the data in order through the REST API.

## Using the Admin Interface
Below we will walk through creating a single site, enclosure, and telescope with a single instrument on it, showing the order in which you should create these objects. From there, you should know how to add additional telescopes or instruments as well. All steps are accomplished by clicking on the corresponding model in the `/admin/` endpoint and clicking the "Add a *" button in the upper right corner of that model's page.

1. Add a Site: Bolded fields cannot be left blank, but the most important fields to fill in are the 3-character **code**, and the **latitude** and **longitude** in decimal degrees. The **code** can be any 3 letters, and each site should have a unique code. It might be convenient to use the 3-character code of the airport nearest the site, as long as two different sites would not have the same code. The **elevation**, **Timezone**/**Tz** and **restart** fields are not used by other OCS applications, so they are just there if you need them in your setup.

2. Add an Enclosure: Fill in a 4-character enclosure **code**, and then link it to the **site** you created in step 1. The 4-character enclosure code can be anything of your choice that describes the type of enclosure. For example if you have two enclosures of the same type at the same site, and you are calling the type `dom`, you could use enclosure codes `doma` and `domb`.

3. Add a Telescope: The 4-character **code** should be something descriptive about the telescope, such as its aperture (i.e. `1m4a`/`1m4b` for two 1.4m telescopes). There are several important properties here that are used automatically by the rest of the OCS. The **aperture** field must be the telescopes diameter in meters - this value will be converted into a string **telescope_class** within the location block of a Request. Telescopes with the same aperture can be scheduled together if they have the same instrument class on them as well. The **latitude** and **longitude**, **horizon** limit, positive and negative **hour angle limits**, and radial **zenith blind spot** fields are all used to compute visibility for targets on the telescope, so they should all be filled in for your telescope if applicable. The **minimum slew overhead** and **slew rate** are currently not used to calculate request durations, but in the future we want to move towards supporting this as an option so that changes in targets within an observing request can have their overheads accurately modeled for a telescope. The **instrument change overhead** is used for computing observation request durations if the instrument is changed between observations in the request.

4. Add a Camera Type: The camera type just represents a class of cameras, which need their **pixels x/y** and **pixel scale** defined, as well as a **code** which should be descriptive of the class of cameras it represents (perhaps some combination of the make/model of the camera).

5. Add Optical Elements: You will need to add an optical element for each specific filter, slit, grating, diffuser, etc. that is an option for your instruments. We will group them into categories in the next step, but for now just add each specific one, with a **code** that users will use to specify that element, and a human readable **name** that will be displayed in the user interface.

6. Add Optical Element Groups: For each category of **Optical Elements** that you created above, such as filters, slits, etc., you will make one Optical Element Group representing that category. The **type** field contains the pluralized category (it **must** end with an `s`), like *filters* or *slits*, while the **name** can be used to give some identifying information to this group. Then select one or more **optical elements** to be a part of this group, and specify an **optical element change overhead** in seconds to be used when computing observation request durations. The user will specify the singular version of this optical element type when setting it to a value within their Requests, i.e. *filter* and *slit*.

7. Add a Camera: You must select the **Camera Type** you created above for this camera, as well as give it a unique **code** and default **orientation** of the camera CCD. You will also select zero or more **optical element groups** that apply to this camera. These will be used for automatic validation of observing requests on this camera.

8. Add an Instrument Category: The category will be reported by the Observation Portal in its description of the available instruments, and can be used by the frontend. Las Cumbres Observatory uses two categories, *IMAGE* and *SPECTRA*, which broadly represent imager and spectrograph type cameras. Our frontend uses this information to provide specific behaviour based on if an instrument is a spectrograph or imager.

9. Add Configuration Types: These are simple a code and human readable string pair for each type of configuration you want your system to support. Some examples supported in the OCS are **EXPOSE**, **REPEAT_EXPOSE**, **SPECTRUM**, **BIAS**, **DARK**. These types usually denote different behaviour that the Telescope Control System may need to use to take the requested observation.

10. Add an Instrument Type: Each type of instrument must have a corresponding **Instrument Type** created to specify its instrument level overheads and default **acceptability threshold** and **configuration type** for that instrument.

11. Add Configuration Type Properties: For each **Configuration Type** that you want to support on an **Instrument Type**, add a **Configuration Type Properties** linking the two and set a corresponding overhead for changing to this configuration type. Also set boolean values of if this type should be **schedulable**, if it should **force acquisition off** (For certain calibration types that don't need acquisition), and if it **requires optical elements** (For certain calibration types that don't need any element set like darks).

12. Add some Mode Types: These are used for creating **Generic Mode Groups**. The Observation Portal expects to validate modes of these specific types, so at a minimum these should be added: **readout**, **guiding**, **acquisition**. If you use instruments that allow configurable rotation, you can also add **rotator** as a mode type, which is also used automatically for validation. You can create other mode types if you have other modes that you will add custom validation to your Observation Portal for.

13. Add Generic Modes: Adding a generic mode of any type requires setting its **code**, its human-readable **name**, and then setting if it should be **schedulable** and specifying its **overhead** in seconds. Where the overhead is applied depends on which of the default mode types this mode will be used for. The cerberus validation schema is optional, but will be used during validation if set.

14. Add Generic Mode Groups: Once you have some **Generic Modes** of a certain mode **type**, you can create a group to apply those modes to a specific **Instrument Type**. One or more modes can be in the group, and an optional default can be specified to be applied if the user does not specify a mode of that type.

15. Add an Instrument: You must specify a **code**, which will be used to specify the instrument an observation is scheduled to take data on. The only automatic **state** used is *SCHEDULABLE*, which means the instrument is available for scheduling. You must specify which **telescope** the instrument is on, and give it one or more **science cameras** that make up the instrument, as well as one **autoguider camera** (the autoguider camera can be one of the science cameras).

This is a lengthy process for the initial setup, but once that is done you should only need to add more instruments or modes or tweak overhead times, which should take considerably less time.

## Using the REST API
ConfigDB can also be setup by POSTing data through the REST API in the proper order. ConfigDB is meant to be run internally (not publicly accessible) so by default it is setup so any authenticated account can POST to the API. The order in which data structures are submitted to the API should follow the same ordering as setting them up via the Admin interface described above. The actual payloads will mostly match the [Configdb API]({% link api/configdb.md %}), with some notable exceptions we will detail below.

* **GenericModeGroups** can take *either* a list of `modes` each containing **GenericMode** data to create or use, *or* a list of `mode_ids` of already created **GenericMode** instances to link to it. You can either create the `modes` along with the group, or you can assign previously created `mode_ids`.
```json
{
    "type": "readout",
    "instrument_type": 21,
    "modes": [{
        "name": "Mode 1",
        "code": "readout_mode_1",
        "overhead": 24.0,
        "schedulable": true,
        "validation_schema": {}
    }],
    "mode_ids": [228, 17, 112],
    "default": "readout_mode_1"
}
```

* **OpticalElementGroups** can take *either* a list of `optical_elements` each containing **OpticalElement** data to create or use, *or* a list of `optical_element_ids` of already existing **OpticalElement** instances to link to it. You can either create the `optical_elements` along with the group, or you can assign previously created `optical_element_ids`.
```json
{
    "name": "my filter group 1",
    "type": "filters",
    "element_change_overhead": 0.0,
    "optical_elements": [{
        "name": "Red",
        "code": "r",
        "schedulable": true,
    }],
    "optical_element_ids": [228, 17, 112],
    "default": "r"
}
```

* **InstrumentTypes** should have their generic `mode_types` set using the **GenericModeGroup** create API. **ConfigurationTypeProperties** can be set through this API but the **ConfigurationTypes** themselves should be created in advance. You can alternatively create the **InstrumentType** without any **ConfigurationTypeProperties** and then create those later through their create API.
```json
{
    "name": "Imager Instrument 1",
    "code": "Imager-01",
    "fixed_overhead_per_exposure": 0.0,
    "instrument_category": "IMAGE",
    "observation_front_padding": 0.0,
    "acquire_exposure_time": 0.0,
    "default_configuration_type": "EXPOSE",
    "default_acceptability_threshold": 90,
    "config_front_padding": 0.0,
    "allow_self_guiding": true,
    "configuration_types": [{
        "configuration_type": "EXPOSE",
        "config_change_overhead": 0.0,
        "schedulable": true,
        "force_acquisition_off": true,
        "requires_optical_elements": true,
        "validation_schema": {}
    }],
    "validation_schema": {},
}
```

See the [Observation Portal Setup]({% link deployment/obs_portal_setup.md %}) section for information on how to set up a new Observation Portal to begin accepting Observation Requests.
