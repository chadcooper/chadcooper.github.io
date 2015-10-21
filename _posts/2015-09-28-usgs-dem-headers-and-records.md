---
layout: post
title: Deciphering a USGS DEM file 
excerpt: The USGS DEM file format is an interesting beast, under the hood.
---

I came across this in Evernote and figured I'd share it in case some poor 
soul should ever have to dig into the guts of the DEM file format like I had 
to a few years ago. As you can see below (click on image for a PDF), it's 
a pretty cryptic and nasty mess in there. 

The DEM I used was derived from [GTOPO 1km elevation data](https://lta.cr.usgs.gov/GTOPO30). 
The source data was converted to a tiff in ArcGIS, then converted to a DEM using the following 
GDAL command:

{% highlight bash %}
D:\Data\GTOPO>gdal_translate -of USGSDEM raster30.tif raster30.dem
{% endhighlight %}

I used the [National Map - Standards for Digital Elevation Models](http://nationalmap.gov/standards/pdf/1DEM0897.PDF) 
as a reference to figure all of this out.

[![_config.yml]({{ site.baseurl }}/images/2015-09-28-dem-header-record.png)]({{ site.url }}/assets/USGSDEM_Header_Definitions.pdf)