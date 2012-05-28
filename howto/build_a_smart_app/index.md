---
layout: page
title: HOWTO Build a SMART App
tagline: first steps
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

This document is a complete SMART-App-Building walk-through. You should first read the [Main Page](../../) 

#ScreenCast


We are re-recording the screencast to catch up with the latest API old [screencast](http://vimeo.com/20113823).

<iframe src="http://player.vimeo.com/video/20113823" width="500" height="375" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen> </iframe>


#Setting up your Environment


A SMART app is a web application that is loaded in an IFRAME hosted by a SMART container. That means you need to (a) write a web app, and (b) connect it to a SMART container. 

You can choose any toolkit you want to write a web app: Java Spring, Ruby on Rails, Python/Django, etc. For the purposes of this documentation, we've chosen webpy, a very simple, minimalist Python web framework, which helps us show you the important SMART-related code more quickly. Also, if you want to get going quickly with the more advanced app features, you probably want to stick with Java or Python for now, as those are the two programming languages in which we've built client libraries. That said, if you're comfortable with OAuth and REST, you can use another programming language without fear. 

We also provide you with a SMART EMR hosted at `sandbox.smartplatforms.org`. We call it the SMART Reference EMR, and we've loaded it with 50 patient records on which you can try out your app. To get going, you'll need to: 

 <ol>
            <li>Navigate to the [developers' sandbox](http://sandbox.smartplatforms.org/login")</li>
            <li>If you haven't done so already, create an account, otherwise just log back in </li>
            <li>Select a patient </li>
            <li>Run the app called &quot;My App&quot; </li>
          </ol>



This will open a SMART app iframe pointing to `localhost:8000`, which is where your app should be running. If you need an app with a different hostname (say, my_internal_server.net), just e-mail joshua dot mandel at childrens.harvard.edu with a manifest file and we'll set you up! 

#Barebones App

Your app needs to serve at least the following URL:

   * test [http://localhost:8000/smartapp/index.html](http://localhost:8000/smartapp/index.html)

You could set up Apache to serve these as static files. In this documentation, we're using webpy for everything, just for consistency. Also, you may find that, for putting up a couple of static files, it's easier to get going with webpy than with Apache. 


##Index

The index file, served at [http://localhost:8000/smartapp/index.html](http://localhost:8000/smartapp/index.html) is where all the fun happens! Make sure to include the SMART page script:

	
{% highlight html %}	
<script src="http://sample-apps.smartplatforms.org/framework/smart/scripts/smart-api-client.js"></script>
{% endhighlight  %}		
	

This script serves to connect your HTML page to the SMART JavaScript library.

Once the client-side library has loaded, your index HTML page has access to a SMART JavaScript object that provides some basic context:
 <ul>
            <li>`SMART.user`, which provides the name and ID of the user who launched the app, typically the physician logged into the SMART EMR.</li>
<li>`SMART.record`, which provides the name and ID of the patient whose record is loaded.</li>
</ul>	

For a complete reference of the app context, check out the JavaScript Library reference.

A more complete index file that displays the current patient's name might thus look like: 
{% highlight html %}
	<!DOCTYPE html>
	<html>
	 <head>
	  <script src="http://sample-apps.smartplatforms.org/framework/smart/scripts/smart-api-client.js"></script>
	 </head>
	 <body><h1>Hello <span id="name"></span></h1>
	 
	 <script>
	 SMART.ready(function(){
	   document.getElementById('name').innerHTML = SMART.record.full_name;
	 });
	 </script>
	 </body>
	</html>
{% endhighlight  %}	

#Using the SMART API

At this point, your SMART app is ready to make API calls to obtain health data. Remember that your app is instantiated in an IFRAME for the specific purpose of accessing a single medical record. This means that, from JavaScript, you can request medical data without specifying patient context, because it's already determined by the JavaScript context.

##Asynchronous Calls

Let's load the patient's medications using SMART.MEDICATIONS\_get(). The most important thing you need to know about all SMART JavaScript APIs is that they are asynchronous: you won't get the meds as a result of the SMART.MEDICATIONS\_get() call. Instead, you need to specify callback functions that will be invoked when the results are ready:
{% highlight javascript %}
	SMART.MEDICATIONS_get().success(function(meds) {
	  // do something with those meds
	}).error(function(err) {
	  // handle the error
	});
{% endhighlight  %}		

Why did we design the API this way? Because, in most cases, the SMART container will need to make a call to a server to obtain the requested data. That could take some time, and it would be very unfortunate if your app was forced to block for a couple of seconds. Instead, your app gets control back from the SMART library call almost immediately and is free to display some pretty progress bar or, more substantively, make additional API calls to obtain a few data points in parallel. 


##Data in RDF form

When data becomes available, the SMART framework invokes your callback function, passing it the resulting medications as a parameter. This result is in the form of an SMARTResponse object containing the RDF graph. RDF \(Resource Description Framework\) is an open and flexible approach to modeling all kinds of data in a graph structure. If you haven't used RDF, you should read our Quick Introduction to RDF and SPARQL.

The bottom line is a SMART medication list is an RDF graph that can be easily navigated and queried. For example, if
meds is an RDF graph, then:
{% highlight javascript %}
meds.graph.where("?medication rdf:type sp:Medication")
{% endhighlight  %}		
selects all of "objects" in the graph that have a datatype sp:Medication, where sp stands for [http://smartplatforms.org/ns#](http://smartplatforms.org/ns#), the location of the SMART vocabulary.

Of course, we want more than just the raw "objects," we want their properties, in particular the name of the drug. The following selects the drug names, which are coded-values, and then the value of those coded values, which are the actual drug-name strings:
{% highlight html %}
meds.graph
	.where("?medication rdf:type sp:Medication")
	.where("?medication sp:drugName ?drug_name_code")
	.where("?drug_name_code dcterms:title ?drugname");
{% endhighlight  %}	

This is effectively a JavaScript query on the RDF graph, and it returns a set of JavaScript objects with properties we're interested in, in particular drugname. We can then iterate over the list of returned objects and extract the drugname property for each one:
{% highlight html %}
var med_names = meds.graph
		 .where("?medication rdf:type sp:Medication")
		 .where("?medication sp:drugName ?drug_name_code")
		 .where("?drug_name_code dcterms:title ?drugname");
		 
	 med_names.each(function(i, single_med) {
		 // do something with single_med.drugname
	   });
{% endhighlight  %}	
	   
##The Complete App

So, to display the patient's medications, we set up an HTML list, \<ul>, and we append to it with the name of each drug in our iteration:
{% highlight html %}
	<!DOCTYPE html>
	<html>
	 <head>
	  <script src="http://sample-apps.smartplatforms.org/framework/smart/scripts/smart-api-client.js"></script>
	 </head>
	 <body><h1>Hello <span id="name"></span></h1>
	 
	 <ul id="med_list">
	 </ul>
	 
	 <script>
	 SMART.ready(function(){
	   document.getElementById('name').innerHTML = SMART.record.full_name;
	   SMART.MEDICATIONS_get().success(function(meds) {
		 var med_names = meds.graph
		   .where("?medication rdf:type sp:Medication")
		   .where("?medication sp:drugName ?drug_name_code")
		   .where("?drug_name_code dcterms:title ?drugname");
		 
		 var med_list = document.getElementById('med_list');
		 med_names.each(function(i, single_med) {
		   med_list.innerHTML += "<li> " + single_med.drugname + "</li>";
		 });
	   }).error(function(err) {
			 alert ("An error has occurred");
	   });
	 });
	 </script>
	 </body>
	</html>
{% endhighlight  %}	

And that's it! In a few lines of HTML and JavaScript code, we've got ourselves an app that can request the medications from the current record and display them. 


#What Next?

<ul>
    <li>learn more about SMART Connect with <a href="/smart-docs-testing/howto/got_statins">GotStatins</a>.</li>
    <li>build a more powerful SMART App, learn <a href="/smart-docs-testing/howto/build_a_rest_app">HOWTO Build a SMART App - REST API Calls</a>.</li>
    <li>build SMART Apps that work while the user is offline: <a href="/smart-docs-testing/howto/background_and_helper_apps">HOWTO Build SMART Background and Helper Apps</a>.</li>
</ul>


