# Architecture Overview

The CDCP platform is based on file data.  So you can start with a folder of data (the source data) 
in a specific, simple format, see [https://github.com/cambridge-collection/dl-data-samples](https://github.com/cambridge-collection/dl-data-samples) . We use TEI for item data, 
JSON / HTML for collection data and TIFF for our image data.

That data is the automatically converted in real time into the processed data, which is easy for the 
application to read.

![CDCP Intro data.svg](images%2FCDCP%20Intro%20data.svg)

This is done using our AWS pipeline (see [https://github.com/cambridge-collection/cudl-terraform](https://github.com/cambridge-collection/cudl-terraform)
or you can use our XSLT and build your own pipeline [https://github.com/cambridge-collection/cudl-data-processing-xslt](https://github.com/cambridge-collection/cudl-data-processing-xslt)).

The processed data is read by a number of applications, which provide the functionality 
for the viewer. (NOTE: Currently there is also a small DB, but this will be retired soon).

![CDCP Intro applications (1).svg](images%2FCDCP%20Intro%20applications%20%281%29.svg)

What does the data look like?  It has a simple hierarchy and is easy to read JSON data, with 
item data in TEI. A collection can have multiple items, and items can be in more than one collection.

![Simple CUDL Dependency Graph Example.svg](images%2FSimple%20CUDL%20Dependency%20Graph%20Example.svg)

So you can edit the source data, so have different title and themes, and override the css, 
so by just editing the data and not having to recompile the site you can get lots of different looks,
and different collection and item data too.
![Simple CUDL Dependency Graph Example.svg](images%2FSimple%20CUDL%20Dependency%20Graph%20Example.svg)
![differentdata.png](images%2Fdifferentdata.png)
## Getting Started

Have a look at the cudl-viewer repo: [https://github.com/cambridge-collection/cudl-viewer](https://github.com/cambridge-collection/cudl-viewer)
We still have some work to get an easy all in one build, so you may have a talk to us so we can let you know which branch to use. 
But you should be able to start up with the test data on the main branch.

For more details see: [setup-local-viewer.md](setup-local-viewer.md)

## OPTIONAL Enhancements

We also have a content loader which edits the data in the source folder, allowing non-expert users 
to edit items and collections and these changes are reflected a few seconds later on the processed data + 
linked viewer website. See our [content-editor.md](content-editor.md) section for more info.

![CDCP Intro - Content loader (1).svg](images%2FCDCP%20Intro%20-%20Content%20loader%20%281%29.svg)

We also have an extension to the processing flow to allow automatic generation of transcriptions by using the 
IIIF manifest the platform provides to read into the Transkribus Expert client, and export in TEI 
format.  Code for this is also under our pipeline terraform: [https://github.com/cambridge-collection/cudl-terraform](https://github.com/cambridge-collection/cudl-terraform)

![CDCP Intro - Transkribus.svg](images%2FCDCP%20Intro%20-%20Transkribus.svg)
