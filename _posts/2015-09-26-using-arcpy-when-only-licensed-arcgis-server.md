---
layout: post
title: Importing arcpy explicitly from an ArcGIS Server install 
excerpt: If you have a server that once had ArcGIS for Desktop installed, but now only Server, you may have issues importing arcpy.
---

I had an Amazon EC2 instance with ArcGIS for Server. This server also had 
ArcGIS for Desktop on it, but no licensing for Desktop anymore. I had tested 
some Python scripts while it had a Desktop license, but after the Desktop 
license was gone, my scripts were failing with ``no module named arcpy`` 
errors. 

After some Googling, I ran across [this Esri KB article](http://support.esri.com/es/knowledgebase/techarticles/detail/40838). 
You can get all of the gory details in the KB article, but the gist of it is 
that the .py extension gets associated with whichever version of Python was installed 
last - in my case it was 32-bit Python with the Desktop install. So, when I was 
no longer licensed for Desktop and ran my script, 32-bit Python ran, but it 
couldn't import ``arcpy``. 

The simple solution here is to explicitly call the 64-bit version of Python that 
installs with ArcGIS for Server, like so:

{% highlight bash %}
C:\Users\Administrator>c:\PYTHON27\ArcGISx6410.3\python.exe
Python 2.7.8 (default, Jun 30 2014, 16:08:48) [MSC v.1500 64 bit (AMD64)] on win
32
Type "help", "copyright", "credits" or "license" for more information.
>>> import arcpy
>>>
{% endhighlight %}