---
layout: page
title: Developers Documentation Packaging Applications via SMART Manifest
tagline: for developers
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

#SMART Manifest

Each SMART app should provide a JSON manifest to declare itself to the container. This manifest defines the app's name, ID, a description, and its application mode. 

##Building an UI App?

If you're building a straightforward SMART UI App, here's a template you can use for your manifest file. Just replace the all-caps placeholders with your own values.

{% highlight javascript %}
{
  "name" : "YOUR APP NAME",
  "description" : "BRIEF DESCRIPTION FOR IN-LINE DISPLAY",
  "author" : "YOUR NAME",
  "id" : "CHOOSE-AN-ID@apps.smartplatforms.org",
  "version" : "YOUR.VERSION.NUMBER",

  "mode" : "ui",
  "scope": "record",

  "index" : "http://PATH/TO/YOUR/index.html",
  "icon" : "http://PATH/TO/YOUR/icon.png"
}
{% endhighlight  %}


##Application Modes

A SMART Applications runs in one of two modes:

* UI Apps mode:"ui" 

interact with the user in the context of the SMART container. Several of the sample apps fall in this category: Med List, Problems List, and Got Statins.

* Frame UI Apps mode:"frame_ui" 

can launch and layout multiple apps side-by-side. Apps in this category include: i2b2 EMR View, as well as our Frame UI Sample.

* Background Apps mode:"background" 

perform some function automatically in the background. A sample app in this category is the Surescripts Connector, which automatically gets a Surescripts dispensing history for each patient record and updates the medications list accordingly. 

#Sample Manifest Files

A UI App provides an index file and an icon. Optionally, the manifest file can declare the SMART container version and the API calls that the app needs. For example, the Med List manifest:

{% highlight javascripts %}
{
  "name" : "Med List",
  "description" : "Display medications in a table or timeline view",
  "author" : "Josh Mandel, Children's Hospital Boston",
  "id" : "med-list@apps.smartplatforms.org",
  "version" : ".1a",

  "mode" : "ui",	
  "scope": "record",

  "icon" :  "http://localhost:8001/framework/med_list/icon.png",
  "index": "http://localhost:8001/framework/med_list/index.html",

  "smart_version": "0.4",

  "requires" : {
       "http://smartplatforms.org/terms#Medications": {
         "methods": ["GET"]
       },
       "http://smartplatforms.org/terms#Demographics": {
         "methods": ["GET"]
       }
  }
}
{% endhighlight  %}

A background app needn't provide an icon or activities. For example, the Surescripts Connector provides the following manifest:

{{% highlight javascript %}
{
  "name" : "SMART Connector",
  "description" : "Keeps SMART updated from a SureScripts feed (long-term, runs in background)",
  "author" : "Josh Mandel, Children's Hospital Boston",
  "id" : "smart-connector@apps.smartplatforms.org",
  "version" : ".1a",

  "mode" : "background"
}
{% endhighlight  %}

##Validating Your Manifest

Tha API Verifier app has a facility for validating SMART manifests. You can either launch the app on the sandbox container or in standalone mode from this URL

[API Verifier](http://apiverifier.smartplatforms.org/smartapp/index.html)

