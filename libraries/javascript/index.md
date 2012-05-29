---
layout: page
title: Developers Documentation SMART App Javascript Libraries
tagline: for developers
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

#What is smart-api-client.js?

Every SMART UI app includes an

<ol><li>
index.html loads smart-api-client.js to provide the app's functionality.</li></ol>

This page describes the functionality that including `smart-api-client.js` provides.

##smart-api-client.js

You include this file in every additinal page of your app, e.g. via

{% highlight html %}
 <script src="http://sandbox-dev.smartplatforms.org:8001/framework/smart/scripts/smart-api-client.js"></script>
{% endhighlight  %}

Once this script loads, your app will have access to a javascript variable called SMART, which can be used to interact with the SMART container via the calls described below. 

##Interacting with the SMART Container via javascript

Inside your index.html, you'll need to be sure the SMART library has finished loading before you can use it. Just put your code inside a call to `SMART.ready()` as shown below. Then you're all set to try out the features below!


The javascript SMART object contains some helpful context describing the current SMART container user and patient record

* SMART.user  {id, full_name}
* SMART.record  {id, full_name}
	
	
	
So, for example, to announce the patient's name, you could

{% highlight javascript %}
alert("The current patient is: " + SMART.record.full_name);
{% endhighlight  %}

If you're looking to make some SMART REST calls, you may be interested in using REST authentication tokens, which you can access via

* SMART.credentials.rest_token
* SMART.credentials.rest_sec


Or you can get the complete SMART OAuth header for your app via

<ul><li>SMART.credentials.oauth_header </li></ul>

##Subscribe to notifications from the SMART Container

A container will notify an app when important events occur. Today the SMART API defines three Container-to-app notifications that your app can subscribe to

* "backgrounded" -- the app has been hidden from view</li>
* "foregrounded" -- the app has been restored to view</li>
* "destroyed" -- the app is being shut down </li>

	
Your app can use the "on" directive to take action when a notification arrives. For example

{% highlight javascript %}
SMART.on("foregrounded", function() {
  refreshAllData();
  alert("Thanks for looking again!");
});
{% endhighlight  %}

##Send notifications to the SMART Container

Your app can also send notifications to the container. Today the SMART API defines only a single App-to-Container notification, which allows an app to request additional screen real estate when there's something big to display

{% highlight javascript %}
SMART.notify_host("request_fullscreen");
{% endhighlight  %}

Please keep in mind that these app->container and container-->app notifications are "fire and forget"; they don't provide a callback mechanism.

##Making API Calls

You can also use the SMART javascript object to make [any API call](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_REST_API_Reference) by calling its api_call method, which takes two parameters


* A dictionary of

<ol><li>method HTTP method as string ('GET', 'POST', 'PUT', or 'DELETE')</li>
	<li>url URL to post to, relative to the SMART container's API base</li>
	<li>contentType as string (defaults to 'application/x-www-form-urlencoded')</li>
	<li>data as string (for data other than x-www-form-urlencoded data) or dictionary (for x-www-form-urlencoded data)</li>
</ol>

* A callback function of

<ol><li>contentType string set according to the response header</li>
<li>data string representation of data </li>
</ol>

For example, we could retrieve all medications for the in-context record by calling

{% highlight javascript %}
 SMART.api_call({ 
                   method: 'GET',
                   url: "/records/" + SMART.record.id + "/medications/",
                   data: {}
                },

	        function(response) {
                   alert('data received: ' + response.body);
                });
{% endhighlight  %}

#Convenience wrappers around common API calls

But you shouldn't need to use the raw .api_call method very often, because the SMART javascript object also provides convenience wrappers around common API calls. The functions below all take a callback function of one argument: the RDF graph that holds the response data, parsed from raw RDF/XML via [rdfquery](http://code.google.com/p/rdfquery/).

## SMART.ALLERGIES_get

GETs all allergies from the in-context record 

##SMART.DEMOGRAPHICS_get

GETs all demographics from the in-context record 

##SMART.FULFILLMENTS_get

GETs the fulfillments from the in-context record 

##SMART.LAB_RESULTS_get

GETs all labs from the in-context record 

##SMART.MANIFESTS_get

GETs all SMART App manifests for apps installed in this container 

##SMART.MANIFEST_get

GETs a sing SMART App manifest 

##GETs a sing SMART App manifest 

GETs all medications from the in-context record 

##SMART.NOTES_get

GETs all notse from the in-context record 

##SMART.PROBLEMS_get

GETs all problems from the in-context record 

##SMART.VITAL_SIGNS_get

GETs all problems from the in-context record </pre> 

#Pulling this together with a quick example...

{% highlight javascript %}
SMART.ready(function() {
  alert("Hello, " + SMART.user.full_name);
});
{% endhighlight  %}