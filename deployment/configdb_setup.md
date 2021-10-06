---
layout: page
title: ConfigDB Setup
---

# Configuration Database Initial Setup

When you start up a Configuration Database for the first time, you will need to log in to its admin interface and fill in the initial representation of your observatory. Below we will walk through creating a single site, enclosure, and telescope with a single instrument on it, showing the order in which you should create these objects. From there, you should know how to add additional telescopes or instruments as well. All steps are accomplished by clicking on the corresponding model in the `/admin/` endpoint and clicking the "Add a *" button in the upper right corner of that model's page.

1. Add a Site: Bolded fields cannot be left blank, but the most important fields to fill in are the 3-character **code**, and the **latitude** and **longitude** in decimal degrees. The **code** can be any 3 letters, and each site should have a unique code. It might be convenient to use the 3-character code of the airport nearest the site, as long as two different sites would not have the same code. The **elevation**, **Timezone**/**Tz** and **restart** fields are not used by other OCS applications, so they are just there if you need them in your setup.

2. Add an Enclosure: Fill in a 4-character enclosure **code**, and then link it to the **site** you created in step 1. The 4-character enclosure code can be anything of your choice that describes the type of enclosure. For example if you have two enclosures of the same type at the same site, and you are calling the type `dom`, you could use enclosure codes `doma` and `domb`.

3. Add a Telescope: The 4-character **code** should be something descriptive about the telescope, such as its aperture (i.e. `1m4a`/`1m4b` for two 1.4m telescopes). There are several important properties here that are used automatically by the rest of the OCS. The **latitude** and **longitude**, **horizon** limit, positive and negative **hour angle limits**, and radial **zenith blind spot** fields are all used to compute visibility for targets on the telescope, so they should all be filled in for your telescope if applicable. The **minimum slew overhead** and **slew rate** are currently not used to calculate request durations, but in the future we want to move towards supporting this as an option so that changes in targets within an observing request can have their overheads accurately modeled for a telescope. The **instrument change overhead** is used for computing observation request durations if the instrument is changed between observations in the request.

4. Add a Camera Type: The camera type just represents a class of cameras, which need their **pixels x/y** and **pixel scale** defined, as well as a **code** which should be descriptive of the class of cameras it represents (perhaps some combination of the make/model of the camera).

5. Add Optical Elements: You will need to add an optical element for each specific filter, slit, grating, diffuser, etc. that is an option for your instruments. We will group them into categories in the next step, but for now just add each specific one, with a **code** that users will use to specify that element, and a human readable **name** that will be displayed in the user interface.

6. Add Optical Element Groups: For each category of **Optical Elements** that you created above, such as filters, slits, etc., you will make one Optical Element Group representing that category. The **type** field contains the pluralized category, like *filters* or *slits*, while the **name** can be used to give some identifying information to this group. Then select one or more **optical elements** to be a part of this group, and specify an **optical element change overhead** in seconds to be used when computing observation request durations.

7. Add a Camera: You must select the **Camera Type** you created above for this camera, as well as give it a unique **code** and default **orientation** of the camera CCD. You will also select zero or more **optical element groups** that apply to this camera. These will be used for automatic validation of observing requests on this camera.

8. Add an Instrument Category: The category will be reported by the Observation Portal in its description of the available instruments, and can be used by the frontend. Las Cumbres Observatory uses two categories, *IMAGE* and *SPECTRA*, which broadly represent imager and spectrograph type cameras. Our frontend uses this information to provide specific behaviour based on if an instrument is a spectrograph or imager.

9. Add an Instrument Type: Each type of instrument must have a corresponding **Instrument Type** created to specify its instrument level overheads and default **acceptability threshold** for that instrument.

10. Add some Mode Types: These are used for creating **Generic Mode Groups**. The Observation Portal expects to validate modes of these specific types, so at a minimum these should be added: **readout**, **guiding**, **acquisition**. If you use instruments that allow configurable rotation, you can also add **rotator** as a mode type, which is also used automatically for validation. You can create other mode types if you have other modes that you will add custom validation to your Observation Portal for.

11. Add Generic Modes: Adding a generic mode of any type requires setting its **code**, its human-readable **name**, and then setting if it should be **schedulable** and specifying its **overhead** in seconds. Where the overhead is applied depends on which of the default mode types this mode will be used for. The cerberus validation schema is optional, but will be used during validation if set.

12. Add Generic Mode Groups: Once you have some **Generic Modes** of a certain mode **type**, you can create a group to apply those modes to a specific **Instrument Type**. One or more modes can be in the group, and an optional default can be specified to be applied if the user does not specify a mode of that type.

13. Add an Instrument: You must specify a **code**, which will be used to specify the instrument an observation is scheduled to take data on. The only automatic **state** used is *SCHEDULABLE*, which means the instrument is available for scheduling. You must specify which **telescope** the instrument is on, and give it one or more **science cameras** that make up the instrument, as well as one **autoguider camera** (the autoguider camera can be one of the science cameras).

This is a lengthy process for the initial setup, but once that is done you should only need to add more instruments or modes or tweak overhead times, which should take considerably less time.

See the [Observation Portal Setup]({% link deployment/obs_portal_setup.md %}) section for information on how to set up a new Observation Portal to begin accepting Observation Requests.
