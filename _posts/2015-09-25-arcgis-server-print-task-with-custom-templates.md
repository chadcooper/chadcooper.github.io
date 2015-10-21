---
layout: post
title: Creating a ArcGIS Server Print Task with your custom print templates 
excerpt: Creating a ArcGIS Print Task with your own custom templates is actually pretty simple.
---

Creating a ArcGIS Server Print Task with your own custom templates is a 
relatively easy task. The following instructions are for ArcGIS Server 10.3, but 
would probably work for a few versions back as well.

First, create your templates, which are essentiall just plain ole MXDs, but 
without and data in them. Leave your main data frame named ``Layers``. Put them in a 
folder of their own. 

If you have a multi-machine ArcGIS Server setup, put you need to store your templates
in a share that all servers in your setup can access and that your ArcGIS Server account 
has access to. If you have a multi-server setup, your ArcGIS Server account needs to be 
a domain level account.

In the Server Tools toolbox, use the Export Web Map tool and create the print task. 
set the output type default and folder location with the MXDs. Leave Web Map as JSON blank 
and leave Output File with whatever it defaults to:

![_config.yml]({{ site.baseurl }}/images/2015-09-25-export-web-map.png)

Hit OK, run the tool. Share the results of the Export Web Map task run out as a 
geoprocessing service:

![_config.yml]({{ site.baseurl }}/images/2015-09-25-share-results.png)

This last step is the key to making this work really nicely for you. In ArcGIS Server Manager 
or the server properties, add the print templates folder as a registered folder. This will 
allow you to add/remove/edit print templates and changes with the templates will propogate 
out to the print service. Worst case is you have to bounce your print service, but your shouldn't 
even have to do that. 

![_config.yml]({{ site.baseurl }}/images/2015-09-25-register-folder.png)

You can now add/remove/edit your templates, save them, and then your changes will be live. 
ArcGIS Server recognizes the changes since the folder is registered with AGS.