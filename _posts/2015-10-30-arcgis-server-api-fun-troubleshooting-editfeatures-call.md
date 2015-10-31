---
layout: post
title: ArcGIS Server REST API fun - Troubleshooting an applyEdits call 
excerpt: When working with the ArcGIS Server, examining what your application is passing to the REST API can shed light on problems.
---

Seems like everyday for the past several weeks now I've been working with the 
ArcGIS Server REST API in some form or fashion. The other day, I ran into a problem 
that I had to get a second set of eyes on from a colleague, and he showed me a new 
way to troubleshoot an issue using the REST API. 

I had a Web AppBuilder application with a widget that runs a GP tool and produces graphics that get 
added to the map. The user then has the option of saving those graphics out to a ArcGIS Server 
feature service, with the backend being several featureclasses in ArcSDE. Problem was, 
one of the graphics layers wouldn't save up to its feature service, and was throwing a 
generic ``500`` error. 

The way my colleague approached this was to, while in the web application, watch the network 
traffic, and when we hit the "Save" button that should push the graphics up to the feature service, 
look for the call to ``applyEdits``, and then check out the request body that got sent up 
to ArcGIS Server:

![_config.yml]({{ site.baseurl }}/images/2015-10-30-rest-api-get-applyedits-endpoint-1.png)

Look at the network call details:

![_config.yml]({{ site.baseurl }}/images/2015-10-30-rest-api-get-applyedits-endpoint.png)

Finally, look at the request body:

![_config.yml]({{ site.baseurl }}/images/2015-10-30-rest-api-json-encoded.png)

It's jibberish still, looking like so:

```
f=json&adds=%5B%7B%22geometry%22%3A%7B%22x%22%3A-9051085.2868
%2C%22y%22%3A3482154.2986999973%2C%22spatialReference%22%3A%
7B%22wkid%22%3A102100%2C%22latestWkid%22%3A3857%7D%7D%2C
%22attributes%22%3A%7B%22OID%22%3A1%2C%22AncillaryRole%22%3
Anull%2C%22Enabled%22%3A1%2C%22Subtype%22%3A1%2C%22LifecycleStatus
%22%3A%22ACT%22%2C%22Ownership%22%3A%22SJC%22%2C%22FacilityID
%22%3A%2227937%22%2C%22InstallDate%22%3A%22%22%2C%22Elevation
%22%3Anull%2C%22LocationDescription%22%3A%223602%20SHORE%20DR%22%2C
...
...
...
```

This request is what got sent up to ArcGIS Server as a URL parameter, so it's 
encoded and we need to decode it before we can use it further. Copy everything 
in the request body, go to [URL Encoder/Decoder](http://meyerweb.com/eric/tools/dencoder/),
paste it in, and hit decode. You'll end up with something like so:

```
f=json&adds=[{"geometry":{"x":-9051085.2868,"y":3482154.2986999973,"spatialReference":
{"wkid":102100,"latestWkid":3857}},"attributes":{"OID":1,"AncillaryRole":null,
"Enabled":1,"Subtype":1,"LifecycleStatus":"ACT","Ownership":"SJC","FacilityID":
"27937","InstallDate":"","Elevation":null,"LocationDescription":"3602 SHORE DR",
"WaterType":"POT","Source":"FLD","SourceConversion":"KF 6-11","ProjectType":null,
"ProjectNumber":null,"AssetCode":null,"ABID":null,"LegacyID":null,
"ReferenceID":"108326","Rotation":null,
...
...
...
```

This is a little more useful, but take it one step further and pretty-print the JSON so 
it is even easier to read. Copy the decoded JSON from the URL parameter you just decoded. 
Skip the beginning part of ``f=json&adds=``, you won't need that. Go to 
[JSON Pretty Print](http://jsonprettyprint.com/), paste, then hit Go. If you get a result of 
``null`` back, your JSON is probably invalid. Make sure you left off ``f=json&adds=`` from the 
beginning. You should get back pretty-printed JSON like so:

```
[
  {
    "geometry": {
      "x": -9431085.2868,
      "y": 4900154.2987,
      "spatialReference": {
        "wkid": 102100,
        "latestWkid": 3857
      }
    },
    "attributes": {
      "OID": 1,
      "AncillaryRole": null,
      "Enabled": 1,
      "Subtype": 1,
      "LifecycleStatus": "ACT",
      "Ownership": "",
      "FacilityID": "27937",
      "InstallDate": "",
      "Elevation": null,
      "LocationDescription": "3602 MAIN DR",
      "WaterType": "POT",
      "Source": "FLD",
      "SourceConversion": "KF 6-11",
      "ProjectType": null,
      "ProjectNumber": null,
      "AssetCode": null,
      "ABID": null,
      "LegacyID": null,
      "ReferenceID": "108326",
      "Rotation": null,
      "ContractorDeveloper": null,
      "Acceptance": "",
      "Diameter": null,
      "CalibrationDate": "",
      "Manufacturer": null,
      "SerialNumber": null,
      "Year": null
    }
  },
...
...
...
```

The JSON above represents the graphics features that we are trying to push up to our 
ArcGIS Server feature service. We can now take this JSON and go to the REST endpoint of 
the feature service and try the ``POST`` there:

![_config.yml]({{ site.baseurl }}/images/2015-10-30-rest-api-apply-edits.png)

In our case, we got back a ``500`` still, kinda what we expected.

```
{
 "error": {
  "code": 500,
  "message": "Unable to complete operation.",
 }
}
```

Thanks for the great error message, ArcGIS Server. We tested writing to the feature service 
from a web map in AGOL, and that worked, so it wasn't a permissions thing. I could edit the 
SDE featureclass in ArcMap too.

Next we started looking at the fields to see if there were differences between what the application 
was trying to pass in to ``applyEdits`` and what the actual fields were. No dice, all looked good 
there. 

We then decided to pare down the fields list and only pass in one geometry at the REST endpoint to 
maybe isolate the problem:

```
[
  {
    "geometry": {
      "x": -9431085.2868,
      "y": 4900154.2987,
      "spatialReference": {
        "wkid": 102100,
        "latestWkid": 3857
      }
    },
    "attributes": {
      "OID": 1,
      "AncillaryRole": null,
      "Enabled": 1
    }
  }
]
```

Pretty quickly we realized the issue was a couple of date fields, such as:

```
"InstallDate": "",
```

The application was trying to pass in a empty string to a date field and SDE was not liking that 
one bit. Passing in ``null`` probably would have worked, but we didn't want to change application 
code. 

The final fix? Those fields were not necessary in the web app, so we went into the MXD behind the 
map service that powers the web map feeding the application and turned off any of the date fields, which 
there were three of. We republished the service, and now we can write out to our feature service, as the 
application is no longer using those date fields since they are turned off.

This was a pretty frustrating problem, but turned out to be one great learning experience by 
exposing me to yet another way to use the ArcGIS Server REST API to troubleshoot an issue.




