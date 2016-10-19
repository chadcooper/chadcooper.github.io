---
layout: post
title: Using cURL and Node to access and parse ArcGIS Server REST endpoints
excerpt: Need to get at the JSON representation of a ArcGIS Server map service? cURL and Node can make it pretty easy.
---

For some routine web application configuration, I need all of the fields and aliases for ArcGIS Server 
map service layers. I have some Python that hits the REST endpoint, but I knew there had to be a simple 
way to do this at the command line. Turns out there is.

Getting at the JSON representation of any ArcGIS Server service is pretty easy. For example:

[http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0?f=pjson](http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0?f=pjson)

The key above is the ``f=pjson`` query string parameter, accessible from any REST endpoint 
via a _teeny, tiny_ JSON link at the top left of the layer's page. See it there?

[http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0](http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0)

Click on it, you know you want to. 

You're now looking at the JSON representation of your map service layer. It's 
all in there: extent, fields, domains, capabilities, etc. 

So how do we get out just the parts we need?

# Installation

I can't remember exactly what got me on this track, but something made me think about using 
cURL to fetch the JSON from the REST endpoint - but then how to parse it? I'd used the Node 
command line before to experiment with JSON parsing, so surely Node could work, huh? 

1. Install [cURL](http://curl.haxx.se/)
2. Install [node](https://nodejs.org/en/)

      Googling turned up the node package [json](http://trentm.com/json/), and playing around with 
      it quickly, I realized this was the ticket to parsing the service JSON.

3. Install the ``json`` package:

      ```
      > npm install -g json
      ```
      
# Parsing

The ``json`` package [examples](http://trentm.com/json/#EXAMPLES) section has some great examples, but here 
is what I did. The gist here is you feed the REST endpoint URL to cURL, then take the output from 
cURL and pipe it into ``json`` to parse it.

Using the ``-a`` flag processes the input as an array and lets you pull out parts, such as the 
current version of ArcGIS Server that is running:

   ```
   > curl -s http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0?f=pjson | json -a currentVersion
   10.2
   ```
   
Or, as I was needing, all of the fields in the layer:

   ```
   > curl -s http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0?f=pjson | json -a fields
   [
   {
    "name": "OBJECTID",
    "type": "esriFieldTypeOID",
    "alias": "OBJECTID",
    "domain": null
   },
   {
    "name": "SHAPE",
    "type": "esriFieldTypeGeometry",
    "alias": "SHAPE",
    "domain": null
   },
   ...
   ...
   ]
   ```

Using pipes, you can chain commands together - so you can take the output from one parse and pipe it 
into another to drill further down. Here I'm pulling out just the field name and alias, and using the 
``-d`` flag to delimit the output with a comma:
   
   ```
   > curl -s http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0?f=pjson | json -a fields | json -a name alias -d,
   OBJECTID,OBJECTID
   SHAPE,SHAPE
   FACILITYID,Facility Identifier
   NAME,Trail Name
   LENGTH,Sidewalk Length
   WIDTH,Sidewalk Width
   SURFTYPE,Surface Type
   HIKING,Hiking Allowed
   MTBCYCLE,MTB Allowed
   ROADCYCLE,Cycling Allowed
   EQUESTRIAN,Horseback Riding Allowed
   SKI,X Country Skiing Allowed
   SNOWMOBILE,Snowmobiling Allowed
   MOTORCYCLE,Motorcycling Allowed
   ATV,ATV'ing Allowed
   MTRVEHICLE,Motorized Vehicles Allowed
   SKILLLEVEL,Skill Level
   CONDITION,Condition
   INSTALLDATE,Install Date
   OWNEDBY,Owned By
   MAINTBY,Managed By
   LASTUPDATE,Last Update Date
   LASTEDITOR,Last Editor
   SHAPE_Length,SHAPE_Length
   ```
   
# Security

The method above works great on open services, but what about secured ArcGIS Server services? 
First, you need to generate a token from ArcGIS Server. Next, add the following cURL flags 
before your REST endpoint URL

* ``-X POST`` - specifies to use the ``POST`` method in the http request. ArcGIS Server requires 
  passing in the token in a ``POST`` and not a ``GET``.
* ``-d token`` - data flag, telling cURL to send the specified data (token, in our case) in the ``POST`` 
  to the http server.

A call ends up looking like so:

   ```
   > curl -X POST -d token=v3Tm_U0YEYFdfxSWH098fsdr42qWgrI2O8BZi-ihavYF-nPE6wA16ai_4oJA9JeMyn  http://server.yourdomain.com/arcgis/rest/services/folder/service/MapServer/0?f=pjson | json -a fields | json -a name alias -d,
   ```
   
# Piping to a text file

Finally, piping output to a text file is simple:

   ```
   > curl -s http://tryitlive.arcgis.com/arcgis/rest/services/Trails/MapServer/0?f=pjson | json -a fields | json -a name alias -d, >output.txt
   ```
   
# https

I just got a new box, and installed curl with Chocolatey. I went to query a REST endpoint on https, and 
kept getting nothing back. Not sure exactly what the root cause was (installation via Chocolatey, something 
I missed installing/config'ing on my new PC, etc.), but I fixed this by following the notes of someone with the 
same issue that was kind enough to post it on the curl Chocolatey page of all places.

1. Download the cacert.pem file from https://curl.haxx.se/docs/caextract.html
2. Track down the actual location of the curl.exe (``C:\ProgramData\chocolatey\bin`` in my case)
3. Put the cacert.pem file beside curl.exe
4. Rename cacert.pem to curl-ca-bundle.crt
5. Restart cmd

# Conclusion

This turned out to be a fun experiment with nice results. It's quick, easy, and requires minimal 
installations. I'd like to explore this further and create a tool that could make this even 
quicker and simpler to use.