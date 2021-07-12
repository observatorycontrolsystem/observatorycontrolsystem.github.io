---
layout: page
title: Configuration Database
subtitle: Django Application that manages Sites, Enclosures, Telescopes, Instruments, and Camera properties
show_sidebar: false
menubar: menu
hide_hero: true
---

# Configuration Database (Configdb)

This Django application manages details about the observatory's sites, enclosures, telescopes, instruments, and cameras. It stores a hierarchical representation of what is available at the observatory, which is used for automatic validation within the other OCS applications. It also contains overheads for different parts of an observation for each instrument, which is used in the [Observation Portal]({{ site.baseurl }}{% link components/observation_portal.md %}) to calculate an estimated duration for each request.

## Structure

![Configdb Models](/assets/images/configdb_models.png)

- **Site** - The top level of the model. It contains a 3 character site code (often the closest airport code), and lat/lon/timezone information
- **Enclosure** - A 4 character enclosure code representing a physical enclosure at a site
- **Telescope** - Contains a 4 character telescope code as well as telescope specific information that aids in calculating target visibility, such as the horizon and hour angle limits and zenith blind spot of the telescope
- **Instrument** - Specific instance of an **Instrument Type** on a **Telescope**, containing one or more science cameras and one autoguider camera and the current observational state
- **Instrument Type** - Generically defines a class of instruments on one telescope aperture size. Contains a lot of details about overheads and properties relating to instruments of this type
- **Instrument Category** - The broad category that this Instrument Type is part of. Usually two categories of Imager and Spectrograph is sufficient to cover most instrument types
- **Configuration Type** - The description of an allowed *type* for a configuration within an Observation Portal observation request. Has some associated properties along with it related to each instrument type
- **Camera** - Specific instance of a **Camera Type** with set of **Optical Elements Groups** associated with it
- **Camera Type** - The details of a class of cameras, such as their pixel scale and resolution
- **Optical Element Group** - A grouping of **Optical Element**s of the same type with an optional default and an element change overhead
- **Optical Element** - A generic code for a single optical path element that can be made available for scheduling or not. This can encapsulate anything in the optical path of the instrument, including filters, slits, diffusers, or gratings
- **Generic Mode Group** - A grouping of **Generic Mode**s of the same type that apply to a single **Instrument Type**, along with an optional default for the group
- **Generic Mode** - A set of properties defining a single generic mode, including overhead, schedulability, and a [Cerberus](https://docs.python-cerberus.org/en/stable/schemas.html) styled validation schema. This can encapsulate any type of modes needed for validation, including but not limited to readout, guiding, acquisition, exposure, and rotator modes

## Special features and Generics

In this section we will highlight a few of the generic aspects of the Configuration Database, which can allow users to customize properties to fit their instruments better.

### Generic Modes

The generic modes and generic mode groupings allow for defining arbitrary *types* of mode sets, which can assist in custom validation within the Observation Portal by overriding corresponding serializers. The default generic mode types used by the base Observation Portal are listed below, but additional types can be added as needed.

| Type | Description |
| ---- | ----------- |
| readout | Validates against the **Instrument Config** `mode` attribute in a **Request**. A named readout mode should be defined and interpreted by the telescope control software to encapsulate any number of instrument parameters, including things like binning |
| acquisition | Validates against the **Acquisition Config** `mode` in a **Request**. A named acquisition mode should be defined and interpreted by the telscope control software to encapsulte different types of acquisition. If additional properties need to be user specified, they can be required within the validation schema of the mode |
| guiding | Validates against the **Guiding Config** `mode` in a **Request**. A named guiding mode should be defined and interpreted by the telscope control software to encapsulte different methods of guiding. If additional properties need to be user specified, they can be required within the validation schema of the mode |
| rotator | Validates against the **Instrument Config** `rotator_mode` in a **Request**. A named acquisition mode should be defined and interpreted by the telscope control software to encapsulte different types of acquisition. If additional properties need to be user specified, they can be required within the validation schema of the mode |

### Optical Elements

Much like the generic mode groups, the optical element groups allow for defining arbitrary *types* of sets of optical elements for a **Camera**. The optical element groups have an element change overhead that is factored into observation request durations, and the elements themselves can be marked as schedulable or not. The intention of generic optical elements is to support anything user configurable that is in the optical path of an instrument, such as filters, slits, diffusers, or gratings. They are used for automatic validation for observation requests submitted in the Observation Portal.

### Cerberus Validation Schema

The **Instrument Type** and **Generic Mode** models currently contain a field called `validation_schema`, which accepts a [Cerberus validation schema](https://docs.python-cerberus.org/en/stable/schemas.html) as a JSON blob. This validation schema is applied to the section of the **Request** in which the mode or instrument_type applies. Validation schemas can be used to enfore that existing fields or extra parameters are specified, or even fill in default values for certain fields that are left blank in the **Request**. They can enforce the typing and valid ranges of fields as well. An example validation schema for a readout mode that sets the binning into the extra_params section is defined below.

```json
{
    "extra_params": {
        "type": "dict",
        "schema": {
            "binning": {
                "type": "integer",
                "allowed": [
                    1
                ],
                "default": 1
            }
        },
        "default": {},
        "required": true
    }
}
```
