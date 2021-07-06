---
layout: page
title: Science Archive
subtitle: Django Application that manages Data products that are stored in AWS S3 buckets
show_sidebar: false
menubar: menu
hide_hero: true
---


# Science Archive

This Django application manages the observatory's data products, and references the raw data products which are stored in AWS S3. It features a REST API for retrieving metadata and download links for data products, and also for submitting metadata and data products for upload to S3. For the complete list of filterable FITS header values, check out the [API specification]({{ site.baseurl }}{% link api/downtime.md %}).

## Ingester Library

This python library is available on PyPI as [ocs_ingester](https://pypi.org/project/ocs-ingester/) is used to upload data products to an AWS S3 bucket, and to update the **Science Archive** with metadata for that data product. It validates that the metadata contains all the required values, and adds in values for which it can determine a default value. It can accept data products of any file type, but if it is not a FITS file, the proper FITS header data will need to be supplied as a dictionary along with the file so the file is queryable.
