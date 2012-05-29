---
layout: page
title: Developers Documentation SMART Container Javascript Libraries
tagline: for developers
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

#What is smart-api-container.js ?

smart-api-container.js enables a SMART container to talk with SMART apps, establishing inter-frame messaging to pass data and notifications between application and container.

#smart-api-container.js is for SMART container developers, not SMART app developers!

smart-api-container.js is used by SMART containers such as electronic medical records platforms. It is a key component that helps turn ordinary EMRs into SMART containers. If you're a SMART app developer, you don't need to worry about the api-container, since it sits on the other side of the interface between your SMART Connect app and the data it consumes.

#Using smart-api-container.js

There are two broad steps involved in turning your ordinary EMR into a SMART container

    Expose the SMART REST API on your server
    Offer the SMART Connect API on your Web site 

This document focuses on #2.

##Including the javascript files

SMART containers the provide SMART Conenct API on their Web site by including (or otherwise loading) three key javascript files: jQuery, jschannel, and smart-api-container. The simplest way is to include the following script tags in your HTML

{% highlight html %}
<script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script> 
<script src="http://sandbox.smartplatforms.org/static/smart_common/resources/jschannel.js"></script> 
<script src="http://sandbox.smartplatforms.org/static/smart_common/resources/smart-api-container.js"></script> 
{% endhighlight  %}


If your existing JavaScript codebase employs the '$' variable and you don't want jQuery to clobber that namespace, you might also want to call

{% highlight html %}
<script type="text/javascript">jQuery.noConflict();</script>
{% endhighlight  %}

##Exposing the SMART Connect Interface
##Instantiate a SMART\_CONNECT\_HOST

smart-api-container attempts to handle the generic messaging and data routing that SMART Connect apps require. To get things started, first instantiate a SMART\_CONNECT\_HOST object. By convention, we'll call it SMART\_HOST

{% highlight javascript %}
   	var SMART_HOST = new SMART_CONNECT_HOST();
{% endhighlight  %}

##Override SMART\_HOST.get\_credentials(app\_instance, callback)

get_credentials is called automatically when a new app launches. Its job is to provide the fledgling app with SMART REST tokens, formatted as an App Instances Credential object (details below).

<b>Input</b>

    App Instance object with UUID, context (details below) 

<b>Callback with</b>

    App Instance Credentials object (details below) 

##Override SMART\_HOST.get\_iframe(app\_instance, callback)

get\_credentials is called automatically when a new app launches -- right after get_credentials. Its job is to provide the fledgling app with an empty iframe DOM element in which to render.

<b>Input</b>

    App Instance object with UUID, context, credentials (details below) 

<b>Callback with</b>

    iframe DOM ojbect 

##Override SMART\_HOST.handle\_api(app\_instance, api\_call, callback\_success, callback\_error)

handle_api is called whenever an already-running app needs to make a API call with SMART CONNECT. Its job is to obtain the results of the specified API call and return them back to the app.

<b>Inputs</b>

    App Instance object with UUID, context, credentials (details below).
    API Call object with method, func, params (details below) 

<b>Callback on success with</b>

{% highlight javascript %}
    callback_success({'contentType': string, data: string}) 
{% endhighlight  %}

<b>Callback on error with:</b>

{% highlight javascript %}
    callback_error(http_status, {'contentType': string, data: string}) 
{% endhighlight  %}

{% highlight javascript %}
Note: your SMART Container may override additional functions if needed. Four functions are provided as hooks for this purpose: SMART_HOST.on_app_launch_begin, SMART_HOST.on_app_launch_complete, SMART_HOST.on_app_launch_delegated_begin, SMART_HOST.on_app_launch_delegated_complete. The framework calls these functions at the beginning and end of launch (and delegated launch) operations
{% endhighlight  %}

##Launch an app

Your Container should provide some way for a user to trigger the launch of an app, for instance by clicking on the app's icon in a sidebar. When this occurs, you'll need to notify the SMART\_CONNECT\_HOST that an app launch has been requested. This will instigate the app launch process, beginning with the creation of an app\_instance with a unique ID. The SMART\_CONNECT\_HOST uses app\_instance objects to keep track of the currently-running apps.

{% highlight javascript %}
SMART_HOST.launch_app(manifest, context, options); Inputs:

    SMART Manifest JSON (see below)
    App Instance Context object (see below)
    JSON object the container may optionally attach to the new app instance 
{% endhighlight  %}

##Subscribe to app notifications

Apps that you've launch may send notifications and requests. For example, an app that desires more screen real estate may sent a "request\_fullscreen" notification. To subscribe to a notification, call SMART\_HOST.on as in the example below

{% highlight javascript %}
SMART_HOST.on("request_fullscreen", function(app_instance) {
   $(app_instance.iframe)
      .css({
          position: 'fixed', 
          width: '100%', 
          height: '100%', 
          left:0, 
          top:0 });
});
{% endhighlight  %}

##Send notifications to an app

When important events occur, you should notify any running apps.

    "backgrounded" when an app instance is hidden from view
    "foregrounded" when an app instance is restored to view
    "destroyed" when an app instance is permanently closed 

For example, if you permit a user to hide an app, you should call: SMART\_HOST.notify\_app(app_instance, "backgrounded");

And if you restore it to view you should call SMART\_HOST.notify\_app(app_instance, "foregrounded");

##Closing apps when the patient record context changes

The SMART Connect API works in a "one patient record at a time" paradigm: that is, within a given browser window, there is one "current patient record" in context, and all running apps share that context. When the patient record context changes, the set of running apps is effectively cleared by calling:

SMART\_HOST.record\_context\_changed();

This will close all currently-running apps -- so apps never have to deal with a context change.

You may want your container to automatically launch new apps after the record context changes . In this case, you can trigger the appropriate logic by defining a handle\_context\_changed function:

{% highlight javascript %}

SMART_HOST.handle_context_changed = function(){
   // your code here
};
{% endhighlight  %}

##Understanding App Instance, Manifest, and API Call JavaScript objects

##App Instance Object
{% highlight javascript %}

The SMART_CONNECT_HOST interface uses plain-old JavaScript objects to represent app instances as follows:


{
  uuid: "string",  // unique ID for this instance of the app (new with each launch.)

  manifest:  // <SMART App Manifest JSON structure>, includes app ID, URL, etc. (see below).

  iframe:  <iframe>, // DOM iframe element in which the app instance should render.

  context:  {  // UI apps need user and patient context; Frame UI apps need user context.

 	user: {
          id: "string",	// User ID assigned by the SMART Container
          full_name: "string" // Flattened string representation of the user's name
        },

        record: {
          id: "string",	// Patient Record ID assigned by the SMART Container
          full_name: "string" // Flattened string representation of the patient's name
        }

  
  
  },

  credentials: // <SMART Credentials JSON structure>, see below

}
{% endhighlight %}

##App Instance Credentials

To support REST apps, your SMART Container should generate OAuth tokens each time an app launches. The OAuth tokens are provided to the app as part of a credentials JavaScript object, which is automatically incorporated into the app_instance object. The credentials object includes

{% highlight javascript %}

{
   api_base: "string", // Base URL for the container's SMART API
   rest_token: "string", // SMART REST Token bound to this user/patient/session
   rest_secret: "string", // SMART REST Secret bound to this user/patient/session
   oauth_header: "string", // OAuth header string embedding context & tokens  (see below)
 }
{% endhighlight  %}

The oauth\_header field is particularly important, since it's sent to the app automatically, for a one-step way for the app to obtain access to the in-context record. The oauth_header is a string representing a well-formed OAuth header, which means that it must supply

{% highlight javascript %}

    oauth_nonce: a one-time value that will not be sent again to this app
    oauth_timestamp: current UNIX epoch time
    oauth_signature_method: "HMAC-SHA1"
    oauth_version: "1.0"
    oauth_consumer_key: the consumer key that your SMART container has assigned to the app being launched
    oauth_signature: a computed signature for the HTTP GET of this app's index.html 
{% endhighlight  %}

The following SMART-specific fields are also required, to provide the launching app with necessary context

{% highlight javascript %}
    smart_app_id: the ID of the app being launched (usually the same as the app's OAuth consumer key)
    smart_record_id: the ID of the patient record on which the app is being launched (should match context.record.id)
    smart_user_id: the ID of the user launching the app (should match context.user.id)
    smart_container_api_base: the base REST URL of the SMART container launching the app (no trailing slash)
    smart_oauth_token: an OAuth token that the app can use to sign requests for the current session
    smart_oauth_token_secret: an OAuth secret that the app can use to sign requests for the current session
{% endhighlight  %}

Here's an example of a fully-formed oauth_header, with line breaks inserted for clarity:

{% highlight javascript %}
'oauth_header' : 'OAuth realm="", 
smart_record_id="1768562",
smart_app_id="problem-list%40apps.smartplatforms.org", 
smart_user_id="joshmandel%40smart.org", 
smart_oauth_token_secret="GHY2zhTL6oG1XLwvWHRB",
smart_oauth_token="Iet6gX4NMbPHjFYBhLkm",
smart_container_api_base="http%3A%2F%2Fsandbox-api.smartplatforms.org",
oauth_signature="QMVJcONlB/O53UUNTpkySuvT+Og%3D",
oauth_nonce="YiAs73cBx7QSO69bpLvh", 
oauth_timestamp="1305921820",
oauth_signature_method="HMAC-SHA1", 
oauth_version="1.0", 
oauth_consumer_key="problem-list%40apps.smartplatforms.org"'
{% endhighlight  %}

##API Call Object

When an app makes an API Call, your handler function will be invoked with an argument that looks like

{% highlight javascript %}

{
 type:  "string", // HTTP method (e.g. "GET")
 func: "string", // URL relative to container base (e.g. "/apps/manifests")
 contentType: "string", // sent to server (e.g. "application/x-www-form-urlencoded")
 params:  <object> // JS Object containing key/value URL parameters
}
{% endhighlight  %}
You can use this object to determine how to respond appropriately.

###SMART Manifest Object

You'll provide the SMART\_CONNECT\_HOST with details about an app to launch by passing a JavaScript manifest object that looks like the one below. For more details, see App Manifest Documentation.
{% highlight javascript %}
{
  "name" : "Med List",
  "description" : "Display medications in a table or timeline view",
  "author" : "Josh Mandel, Children's Hospital Boston",
  "id" : "med-list@apps.smartplatforms.org",
  "version" : ".1a",

  "mode" : "ui",	
  "scope": "record",
  "icon" :  "http://app-server/framework/med_list/icon.png",
  "index": "http://app-server/framework/med_list/index.html"
}
{% endhighlight  %}

###A "working" example

Here's a complete example of a SMART Container. This container implements only one API call: (GET medications), and displays an alert if the contained app attempts to call any other function.

You can view the example live at [1] -- and be sure to view the source code!

##Some manifests online

If you're building a container, here are some manifests you can try loading to get started, hosted in our sandbox

  [http://sample-apps.smartplatforms.org/framework/got_statins/smart_manifest.json](http://sample-apps.smartplatforms.org/framework/got_statins/smart_manifest.json)<br>
  
[http://sample-apps.smartplatforms.org/framework/cardio_risk_viz/smart_manifest.json](http://sample-apps.smartplatforms.org/framework/cardio_risk_viz/smart_manifest.json)<br>

[http://sample-apps.smartplatforms.org/framework/med_list/smart_manifest.json](http://sample-apps.smartplatforms.org/framework/med_list/smart_manifest.json)<br>

[http://sample-apps.smartplatforms.org/framework/problem_list/smart_manifest.json](http://sample-apps.smartplatforms.org/framework/problem_list/smart_manifest.json)<br>
    
