---
layout: page
title: HOWTO Build SMART Background and Helper Apps
tagline: third steps
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

##Background Apps

<img src="http://wiki.chip.org/smart-project/images/d/d7/Background-app.png" style="float: right">

So far, we have talked about SMART apps that access medical record data on behalf of an active user of the EMR/PCHR/data-mining platform. What about apps that autonomously take action when no one is logged in? An app might want to check every medical record for a particular datapoint (e.g. a recalled drug). The SMART Platform supports this background app use case as follows:
<ul>
	<li>the app is specially authorized to loop through every known record in the system</li>
    <li>on each looped record, the app obtains OAuth credentials necessary to access that one record.</li>
    <li>for each record, the app can access the SMART REST API as it normally would, using the credentials obtained for that record. </li>
	</ul>
	
Importantly, a background app is not necessarily a web app, since it may not have a user-facing web interface. In this example, we will demonstrate just that: a simple Python program that acts as a SMART background app, but does not respond to any web requests. So, if you've figured out how to build a SMART REST app, all you need to know now is how to obtain the credentials needed to cycle through all the SMART Container's medical records.

The first thing you need is, of course, a SmartClient instance to make REST API calls against the SMART container. If you don't have that library running yet, check out the SMART Python Client. Then

{% highlight html %}
  smart_client = smart.SmartClient(
    'smart-background-app@apps.smartplatforms.org',
    {'api_base' : 'http://sandbox-api.smartplatforms.org'},
    {'consumer_key' : 'smart-background-app@apps.smartplatforms.org', 'consumer_secret' : 'smartapp-secret'},
     None)
{% endhighlight  %}


The important difference between this SmartClient and the one you created for use in SMART REST API calls is that this one has no resource-specific credentials. That's because, at first, you're not going to make API calls specific to any single medical record. In OAuth parlance, this is called a "2-legged call." Now, it's time to iterate through all records available at that SMART Container. The specific API calls to do this can be found in our REST API, but the SMART Python client library makes it easy for you: 

{% highlight html %}
for record_id in smart_client.loop_over_records():
   # do stuff with each record
{% endhighlight  %}

Note that, in each iteration, the smart_client is modified to automatically use the credentials needed to access this specific record. If you were to call the API manually, you would have to extract the tokens you get from the container, and insert them into the smart_client object.

So, using a simple query to read drug names: 

{% highlight html %}
 QUERY = """
         PREFIX dcterms:<http://purl.org/dc/terms/>
         PREFIX sp:<http://smartplatforms.org/terms#>
         PREFIX rdf:<http://www.w3.org/1999/02/22-rdf-syntax-ns#>
         SELECT  ?drugname
         WHERE {
            ?med rdf:type sp:Medication .
            ?med sp:drugName ?drugname_code .
            ?drugname_code dcterms:title ?drugname .
         }
         """
{% endhighlight  %}

We can simply iterate through the records, retrieve each record's medications, and print them to the screen: 

{% highlight html %}
  for record_id in smart_client.loop_over_records():
    medications = smart_client.records_X_medications_GET(record_id = record_id)
    med_names = medications.graph.query(QUERY)
    print "%s: %s" % (record_id, ", ".join([str(m) for m in med_names]))
{% endhighlight  %}

#Helper Apps	

You now know how to build simple SMART apps in HTML and JavaScript, SMART apps that make REST API calls, and SMART apps that run entirely in the background without a UI. What about apps that provide services to other apps? These we call Helper Apps.

For the SMART App Challenge, Helper Apps are not yet implementable. Expect this documentation to be updated summer 2011. 