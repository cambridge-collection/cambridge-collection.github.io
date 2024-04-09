# TEI Data Processing Overview

All outputs generated from TEI source files are created by our TEI Data Processing pipeline (<https://github.com/cambridge-collection/cudl-data-processing-xslt>).

The pipeline is dockerised. It can be run in an AWS lambda, locally or within a CI/CD system. It uses Apache Ant (<https://ant.apache.org/>) to run the necessary XSLT transforms to create the output files.

## Overview of the phases of the data build

The build contains three phases:

![The TEI Data Processing Pipeline](images%2Ftei-data-processing.svg)

1. Paginate the TEI source file and generate TEI P5 XML and HTML files for every page that contains *significant* content.
2. Consolidate and process the metadata into an XML file encoded according to the <https://www.w3.org/TR/xpath-functions-3/#schema-for-json> schema.
3. Generate derivative data views from the consolidated data file.

The default build runs all three phases on the specified TEI file. It is also possible to call each phase independently, provided the necessary cached data files are available.

### Pagination of the TEI Source file

The pagination process creates TEI P5 XML and html files for every page that contains *significant* content (*i.e.* textual content or certain significant empty elements, like `<figure/>`, `<g/>`, `<gap/>` that produce output intended for the reader). This ensures that only pages that contain content visible to the end user will be generated.

The TEI P5 page XML files are cached so that they can be made available for end users. They can also be used when updating the html view of the documents so that every item in the collection does not need to be processed in its entirety.

### Consolidation of the Metadata

There are often many different ways to record the same information in TEI. This can make maintaining pipelines that generate multiple outputs from them challenging to maintain.

It can either result in either repeated code fragments that become progressively harder to keep up to date or in the creation of a standardised library of functions/templates to generate the data.

CDCP aims to manage this complexity in two ways. The creation of centralised library for accessing and processing all data makes it easier to ensure that any changes required to reflect new TEI coding styles or improve the output are consolidated in one place.

While a centralised library can help prevent the rise of scissors and paste code, it still poses several challenges when operating at scale over hundreds of thousands of items. Some processes, such as the one that determines whether a page contains content or not, take an appreciable amount of resources to run. Many of these actions would need to be performed for each output file. While the cost/time implications of doing so  aren’t onerous when processing a single document, they are appreciable when run on more complex materials or larger batches of files.

CDCP’s TEI Data Processing pipeline remedies these issues by operating on the principle that raw data should be processed as few times as possible. It generates a XML file, encoded according to the W3C schema for representing JSON, that consolidates all the processed metadata, and page transcriptions and translations for each item in our collection. These consolidated XML files contain the cleaned data necessary to generate all derivative views for our core outputs quickly and efficiently.

### Generation of derivative views for downstream services

The final phase of the process generates the derivative views of the data used by other services within the CDCP ecosystem, like the json that is indexed by SOLR (our search engine) or used by the viewer. Cambridge’s build creates an additional JSON file for use by the pipeline that ingests CDL TEI transcriptions into our Data Preservation system.

The XSLT files required to generate these outputs are small, ranging from 800 bytes to 11K, and do not require any costly data analysis/processing.

### Copy Output to user-specified locations

The outputs created by this pipeline are released once the pipeline’s core actions have been successfully completed. This prevents spurious build artefacts from entering CDCP when some part of the process fails.

The chief outputs for each item are:

* Consolidated Metadata XML file
* TEI P5 XML and HTML files for each page with significant content. If a document does not contain any pages with significant content, no TEI or HTML files will be generated
* Custom JSON containing the metadata, collection information, transcription and translations for indexing by SOLR
* JSON required for the CDCP Viewer
* JSON required by Cambridge University Library’s Digital Preservation pipeline.