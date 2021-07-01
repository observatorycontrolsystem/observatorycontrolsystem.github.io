---
layout: page
title: Science Archive
subtitle: Django Application that manages Data products that are stored in AWS S3 buckets
show_sidebar: false
menubar: menu
hide_hero: true
---


# Science Archive

This django application manages manages the observatory's data products, and references the raw data products which are stored in AWS S3. It features a REST API for querying for the metadata and download links for data products, and for submitting the metadata and S3 location of newly ingested data products. For the complete list of filterable fits header values, check out the [API specification]({{ site.baseurl }}{% link api/downtime.md %}).

## Ingester

This python library `ocs_ingester` is used to upload data products to an AWS S3 bucket, and to update the **Science Archive** with metadata for that data product. It validates that the metadata contains all the required values, and adds in values for which it can determine a default value. It can accept data products of any file type, but if it is not a .fits file, the proper fits header data will need to be supplied as a dictionary along with the file so the file is queryable.
