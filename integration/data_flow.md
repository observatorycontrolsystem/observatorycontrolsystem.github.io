# Observatory Data Flow

For users who wish to utilize the [Science Archive](https://github.com/observatorycontrolsystem/science-archive) for data archival and retrieval, this document describes the components necessary.

## Main Components

* [OCS Ingester](https://github.com/observatorycontrolsystem/ingester) - commonly used as a library to simplify the ingestion process of observatory data products into the science archive. 
* [OCS Science Archive](https://github.com/observatorycontrolsystem/science-archive) - A Django application which aids in the archival storage and retrieval of observatory data products.

Both of these applications make use of a common library, the [OCS Archive Library](https://github.com/observatorycontrolsystem/ocs_archive). The OCS Archive library allows the OCS Ingester and Science Archive to share common configuration and reduces the need for duplicated code across the two projects.


## Configuration

Because both the OCS Science Archive and OCS Ingester depend on the OCS Archive Library, the two can share common configuration. Runtime configuration for the OCS Archive Library is set using environment variables - see [OCS Archive Configuration Documentation](https://github.com/observatorycontrolsystem/ocs_archive#configuration) for more detail about possible configuration options.

## Telescope Control System Integration

In the execution of an observation, the observatory's Telescope Control System generates data products. For these products to be archived and served to users or staff via an OCS Science Archive, they must be ingested.

### A Simple Ingestion Scheme

Once a TCS has created its final data products to be archived, the TCS may call out to the Ingester library directly. If the TCS is written in Python, the call is as simple as:

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

### Persistent Queue for Data Products

_Note: This only applies when a separate process or application is responsible for ingesting data products from the TCS_


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
* **NonFatalDoNotRetryError** - Exception that is raised when an error happens that should not be retried and is also not a fatal condition.
* **DoNotRetryError** - Exception that is raised when an error happens that will undoubtedly repeat if called again. The task should not be retried.

Handling these errors and retrying ingestion if appropriate will increase the robustness of an observatory's data product flow. Appropriate retry logic combined with queueing infrastructure leads to a system which is fault-tolerant and able to recover easily from intermittent service outages.

In Python, the [Tenacity](https://tenacity.readthedocs.io/en/latest/) package provides a set of function decorators that allow for various retry schemes.

### Keeping a Record of Data Product Ingestion

Applications may need to keep track of some of the data products they've ingested into the Science Archive. For instance, a data pipeline that creates calibration frames for use in later science data reduction may need to access these images from the science archive to perform scientific reductions.

One method to keep a record of these data products is using a relational database table that the application can access as needed. When a data product is created by the application, it creates a new image record in its database containing the Science Archive frame ID alongside any other useful metadata that the application may need (filter used, binning, instrument name, etc...).

By storing the frame ID in the record, the lookup process in the Science Archive is greatly sped up, as the Science Archive record for this image can be accessed by simply by sending an HTTP GET request to the `/frames/<image-ID>` endpoint.

For an observatory whose Science Archive is handling many requests, this scheme prevents internal applications from adding unnecessary load to the system by allowing for near-instant access by ID, rather than using more expensive HTTP filter parameters.
