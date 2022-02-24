---
layout: page
title: Science Archive
subtitle: Django Application that manages Data products that are stored in AWS S3 buckets
---

# Science Archive

This Django application manages archival and retrieval of an observatory's data products. It features a REST API for retrieving metadata and download links for data products, and also for submitting metadata and data products for upload to a cloud or local filestore. For the complete list of filterable FITS header values, check out the [API specification]({% link api/science_archive.md %}).

## Ingester Library

This python library is available on PyPI as [ocs_ingester](https://pypi.org/project/ocs-ingester/) is used to upload data products to an AWS S3 bucket, and to update the **Science Archive** with metadata for that data product. It validates that the metadata contains all the required values, and adds in values for which it can determine a default value. It can accept data products of any file type, but if it is not a FITS file, the proper FITS header data will need to be supplied as a dictionary along with the file so the file is queryable.

API documentation for the ocs_ingester is available on [ReadTheDocs](https://ingester.readthedocs.io/en/latest/)

## ocs_archive Library

A base library for the Science Archive and Ingester library to support generalized input file types, generalized data stores, and shared configuration items. Both the Ingester and OCS Science Archive utilize this library, which provides common utility code and configuration for both projects.

More information is available on the [ocs_archive GitHub project page](https://github.com/observatorycontrolsystem/ocs_archive)
