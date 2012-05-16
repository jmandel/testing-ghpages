---
layout: page
title: HOWTO Build a SMART App
tagline: for developers
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

This document is a complete SMART-App-Building walk-through. You should first read the [Main Page](../) 

#ScreenCast


We are re-recording the screencast to catch up with the latest API old [screencast](http://vimeo.com/20113823).


#Setting up your Environment


A SMART app is a web application that is loaded in an IFRAME hosted by a SMART container. That means you need to (a) write a web app, and (b) connect it to a SMART container. 

You can choose any toolkit you want to write a web app: Java Spring, Ruby on Rails, Python/Django, etc. For the purposes of this documentation, we've chosen webpy, a very simple, minimalist Python web framework, which helps us show you the important SMART-related code more quickly. Also, if you want to get going quickly with the more advanced app features, you probably want to stick with Java or Python for now, as those are the two programming languages in which we've built client libraries. That said, if you're comfortable with OAuth and REST, you can use another programming language without fear. 

We also provide you with a SMART EMR hosted at sandbox.smartplatforms.org. We call it the SMART Reference EMR, and we've loaded it with 50 patient records on which you can try out your app. To get going, you'll need to: 

 <ol>
            <li>Navigate to the [developers' sandbox](http://sandbox.smartplatforms.org/login")</li>
            <li>If you haven't done so already, create an account, otherwise just log back in </li>
            <li>Select a patient </li>
            <li>Run the app called &quot;My App&quot; </li>
          </ol>



This will open a SMART app iframe pointing to localhost:8000, which is where your app should be running. If you need an app with a different hostname (say, my_internal_server.net), just e-mail joshua dot mandel at childrens.harvard.edu with a manifest file and we'll set you up! 

#Barebones App

Your app needs to serve at least the following URL:

   <ul>
        <li> [http://localhost:8000/smartapp/index.html](http://localhost:8000/smartapp/index.html) </li>
	</ul>

You could set up Apache to serve these as static files. In this documentation, we're using webpy for everything, just for consistency. Also, you may find that, for putting up a couple of static files, it's easier to get going with webpy than with Apache. 


##Index

The index file, served at [http://localhost:8000/smartapp/index.html](http://localhost:8000/smartapp/index.html) is where all the fun happens! Make sure to include the SMART page script:

	
		
	<script src="http://sample-apps.smartplatforms.org/framework/smart/scripts/smart-api-client.js"></script>
		
	

This script serves to connect your HTML page to the SMART JavaScript library.

Once the client-side library has loaded, your index HTML page has access to a SMART JavaScript object that provides some basic context:
<ul>
<li>
SMART.user, which provides the name and ID of the user who launched the app, typically the physician logged into the SMART EMR.</li>
    <li>SMART.record, which provides the name and ID of the patient whose record is loaded. </li>
	</ul>

For a complete reference of the app context, check out the JavaScript Library reference.

A more complete index file that displays the current patient's name might thus look like: 

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