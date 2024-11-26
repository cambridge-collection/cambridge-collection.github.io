# Setting up a local image server

## Requirements

You will need a IIIF image server setup. We're using the [IIPImage server](https://iipimage.sourceforge.io/) which is installed using the 
following [setup instructions](https://iipimage.sourceforge.io/documentation/getting-started).

You will also need some images. Images can be in either **TIFF** or **JPEG2000** format.

## Setting up the image server

Once you have installed the image server, using the instructions above, you should be able to access images using the
[IIIF image API](https://iiif.io/api/image/2.1/).  We're currently using version 2.1.

Image requests will be in the format: 

    {scheme}://{server}{/prefix}/{identifier}/{region}/{size}/{rotation}/{quality}.{format}

For example: 

    http://www.example.org/image-service/abcd1234/full/full/0/default.jpg

You can also get information about each image in json format using the Image Request URL format:

    {scheme}://{server}{/prefix}/{identifier}/info.json

For example:

    http://www.example.org/image-service/abcd1234/info.json

Once you have setup your image server, and you can receive images using the
IIIF format described above you can proceed to connecting it to the cudl-viewer.

## Connecting images to the viewer

Take a look at the global properties file which is under:
    
[docker/cudl-global.properties](https://github.com/cambridge-collection/cudl-viewer/blob/iiif_images_2023/docker/cudl-global.properties)

This file contains all the properties which vary between dev, staging and live instances. But as 
we're testing locally we can change the values here.  However if you want to package and deploy the viewer
somewhere else you will need to make sure the viewer has a copy of this file somewhere on the classpath.

For the image server we're interested in the following:

    # URLs for image and services servers.
    imageServer=https://images.lib.cam.ac.uk/
    services=http://cudl-services/
    IIIFImageServer=https://images.lib.cam.ac.uk/iiif/

The **imageServer** value is currently used for thumbnail images which appear in the collections
pages and in search results.  The path for each thumbnail image is:

    imageServer + value-for-'thumbnailImageURL'-property-in-item-json-data

The **IIIFImageServer** value is used for the IIIF images in the Mirador viewer and for the main 
zoomable image on the item/document page. 

So to connect to the image server, if you want both your thumbnail images and your main zoomable image to 
use your newly installed IIIF server you need to edit 

    imageServer=YOUR_IMAGE_SERVER
    IIIFImageServer=YOUR_IMAGE_SERVER

and you will need to ensure that the item data in your item json file has the thumbnail URL set for each page to e.g. 

    /abcd1234/full/175,0/0/default.jpg

for an image that is a max of 175 pixels high.

You can then restart the viewer to pick up the images. 