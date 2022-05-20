---
layout: page
title: Observatory Data Flow
---

# Observatory Data Flow

For users who wish to utilize the [Science Archive](https://github.com/observatorycontrolsystem/science-archive) for data archival and retrieval, this document describes the components necessary.

## Main Components

* [OCS Ingester](https://github.com/observatorycontrolsystem/ingester) - commonly used as a library to simplify the ingestion process of observatory data products into the science archive. 
* [OCS Science Archive](https://github.com/observatorycontrolsystem/science-archive) - A Django application which aids in the archival storage and retrieval of observatory data products.

Both of these applications make use of a common library, the [OCS Archive Library](https://github.com/observatorycontrolsystem/ocs_archive). The OCS Archive library allows the OCS Ingester and Science Archive to share common configuration and reduces the need for duplicated code across the two projects - this shared code contains classes used to abstract data files and data storage backends. See the [Advanced Topics](#advanced-topics) section for more information about customizing the behavior of the `ocs_archive` library.


## Configuration

Because both the OCS Science Archive and OCS Ingester depend on the OCS Archive Library, the two can share common configuration. Runtime configuration for the OCS Archive Library is set using environment variables - see [OCS Archive Configuration Documentation](https://github.com/observatorycontrolsystem/ocs_archive#configuration) for more detail about possible configuration options.

## Telescope Control System Integration

In the execution of an observation, the observatory's Telescope Control System generates data products. For these products to be archived and served to users or staff via an OCS Science Archive, they must be ingested.

### A Simple Ingestion Scheme

Once a TCS has created its final data products to be archived, the TCS may call out to the Ingester library directly. If the TCS is written in Python, the call is as simple as setting the relevant environment variables for your filestore type and running:

```python
from ocs_ingester.ingester import upload_file_and_ingest_to_archive

with open('/path/to/file', 'rb') as file:
    ingester_response = upload_file_and_ingest_to_archive(file)

...
# optionally do something with the ingester response
```

The Ingester Response contains information about the archived frame, such as its location (URL for cloud filestore), ID in the Science Archive database and other frame metadata. This can be useful if the application ingesting the data wishes to keep its own record of data products and their location in the archive.

For more detailed information about the OCS Ingester and its capabilities, see the [developer documentation](https://ingester.readthedocs.io/en/latest/index.html).

## Advanced Topics

### Overriding Classes in the `ocs_archive` Library

The `ocs_archive` library provides base classes to represent observatory data products and filestore locations in order to provide a unified interface to various types of data products and filestore solutions.

#### Data Products

Within the `input` module of the `ocs_archive` library there exist classes to represent various different types of files such as FITS, GZipped TAR files, etc...

If an observatory wishes to add their own file specification, they may do so by sub-classing the `DataFile` class and adding any customizations needed. For examples of this functionality, see the [implementation](https://github.com/observatorycontrolsystem/ocs_archive/blob/main/ocs_archive/input/fitsfile.py#L10) of the `FitsFile` class, which customizes the base `DataFile` class to work with FITS files. For Las Cumbres Observatory's purposes, we have further sub-classed the `FitsFile` class into an [`LCOFitsFile`](https://github.com/observatorycontrolsystem/ocs_archive/blob/main/ocs_archive/input/lcofitsfile.py#L16) to add additional functionality to meet the needs of our observatory. 

Once a new file specification has been added, it should be added to the [`FileFactory` class](https://github.com/observatorycontrolsystem/ocs_archive/blob/main/ocs_archive/input/filefactory.py#L14), which is responsible for returning the appropriate `DataFile` subclass given a file's extension. It utilizes a mapping from file extension -> `DataFile` class to determine which class to construct, given an input file:

```python
EXTENSION_TO_FILE_CLASS = {
    '.fits.fz': 'ocs_archive.input.lcofitsfile.LcoFitsFile',
    '.fits': 'ocs_archive.input.fitsfile.FitsFile',
    '.tar.gz': 'ocs_archive.input.tarwithfitsfile.TarWithFitsFile',
    '.pdf': 'ocs_archive.input.file.DataFile'
}
```

This mapping can be overridden using the `FILETYPE_MAPPING_OVERRIDES` environment variable if the default is not sufficient or needs to be changed.


#### Filestore

A set of classes which represent storage backends exists within the `storage` module of the `ocs_archive` library. 

The base [`FileStore` class](https://github.com/observatorycontrolsystem/ocs_archive/blob/main/ocs_archive/storage/filestore.py#L20) is intended to represent a data product storage backend, and can be sub-classed and extended to work with many different storage solutions.

For example, the [`S3Store`](https://github.com/observatorycontrolsystem/ocs_archive/blob/main/ocs_archive/storage/s3store.py#L23) contains customizations to the base `FileStore` class to allow observatory data products to be stored in [Amazon S3](https://aws.amazon.com/s3/). If an observatory needs to use a different cloud provider, then a new subclass can be created. 

The `ocs_archive` also provides a FileStore class, [`FileSystemStore`](https://github.com/observatorycontrolsystem/ocs_archive/blob/main/ocs_archive/storage/filesystemstore.py#L8), made to work with network or local filestores, if an observatory does not wish to store their data with a cloud provider. Filesystem storage does not provide version history or time authenticated generated urls, so S3 Storage should be heavily preferred for production setups.

### Persistent Queue for Data Products

{% include notification.html message="Note: This only applies when a separate process or application is responsible for ingesting data products from the TCS" status="is-info" icon="fas fa-exclamation-triangle" %}

<br/>

When a TCS generates data products to be ingested, it's helpful to use a queueing system such as [RabbitMQ](https://www.rabbitmq.com/) to keep track of images to be ingested. By utilizing a simple queuing architecture, if the application responsible for ingesting images ever experiences downtime, the TCS can still queue images to be ingested and the ingestion application will be able to catch up after its outage.

![Ingester queueing scheme](/assets/images/ingester_queueing_scheme.png)

The queue message can be as simple as the location from which to retrieve the file to be ingested, but should contain the minimum amount of information necessary for the ingestion application to successfully ingest the data to the science archive.

```json
{"filepath": "/path/to/file/on/disk"}
```

### Retry Scheme for Data Product Ingestion

For added robustness, it is good practice to integrate retry logic into the data product ingestion process. It's for this reason that the OCS Ingester defines a set of Python [exception classes](https://github.com/observatorycontrolsystem/ingester/blob/main/ocs_ingester/exceptions.py) that will be raised in appropriate situations:

* **RetryError** - Exception that is raised when an error happens that can be retried.
* **BackoffRetryError** - Exception that is raised when an error happens that can be retried with an expontential backoff. For example, networking latency errors that may succeed at a later time.
* **DoNotRetryError** - Exception that is raised when an error happens that will undoubtedly repeat if called again. The task should not be retried.
* **NonFatalDoNotRetryError** - Exception that is raised when an error happens that should not be retried and is also not a fatal condition. _This is different from the DoNotRetryError as it indicates the failure wasn't a fatal issue in the ingestion of an image (e.g. file doesn't exist on disk or Astropy exception raised), but rather indicates an error such as the file already existing in the filestore._


Handling these errors and retrying ingestion if appropriate will increase the robustness of an observatory's data product flow. Appropriate retry logic combined with queueing infrastructure leads to a system which is fault-tolerant and able to recover easily from intermittent service outages.

In Python, the [Tenacity](https://tenacity.readthedocs.io/en/latest/) package provides a set of function decorators that allow for various retry schemes.

### Keeping a Record of Data Product Ingestion

Applications may need to keep track of some of the data products they've ingested into the Science Archive. For instance, a data pipeline that creates calibration frames for use in later science data reduction may need to access these images from the science archive to perform scientific reductions.

One method to keep a record of these data products is using a relational database table that the application can access as needed. When a data product is created by the application, it creates a new image record in its database containing the Science Archive frame ID alongside any other useful metadata that the application may need (filter used, binning, instrument name, etc...).

By storing the frame ID in the record, the lookup process in the Science Archive is greatly sped up, as the Science Archive record for this image can be accessed by simply by sending an HTTP GET request to the `/frames/<image-ID>` endpoint.

For an observatory whose Science Archive is handling many requests, this scheme prevents internal applications from adding unnecessary load to the system by allowing for near-instant access by ID, rather than using more expensive HTTP filter parameters.
