---
layout: page
title: HOWTO Build a SMART App - REST API Calls
tagline: second steps
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

This document shows you how to build a more advanced SMART App with server-side logic and REST API Calls.

You should first read the [Main Page](../../) and [HOWTO Build a SMART App](../build_a_smart_app). 

#ScreenCast


We are re-recording the screencast to catch up with the latest API changes. Please see the text-based tutorial below, or see the old screencast. 

<iframe src="http://player.vimeo.com/video/20766064?title=0&amp;byline=0&amp;portrait=0" width="400" height="340" frameborder="0" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>


#Server to Server Authentication

When your app was making JavaScript API calls, authentication and authorization were entirely transparent to you, the app builder, because the SMART Container was able to take care of it all: the JavaScript API call simply notified the outer frame of the data request, and the outer-frame knows who the logged-in user is and what medical record they're currently considering.

Now, we consider the case where you want the backend of your app to obtain data directly from the SMART Container, e.g. the SMART Reference EMR, using the REST API. In that case, the SMART Container doesn't know a-priori who is making the call and whether they're authorized to do so. We need to authenticate the call somehow, and you, the app builder, need to know how to ensure that your calls are properly authenticated. 

##OAuth

For this purpose, we use OAuth, an open-standard for access delegation.

OAuth bundles two important features:

<ol>
    <li>a way to label and sign HTTP requests using an identifier token and a secret string</li>
    <li>a dance involving the user's browser, the data server, and the server that wishes to consume the data, which, when the user approves the exchange, provides the data consumer with the token and secret needed to perform the authenticated API calls as per (1).</li>
	</ol>

SMART employs (1), but implements a much simpler approach to (2), providing your app with the requisite token and secret. We'll describe this simpler approach here. At a high-level, your app can simply look for a URL parameter called `oauth_header` set by the SMART JavaScript client library. In this value, you will find the OAuth token and secret you need. 


##Passing Tokens via URL Parameter

The SMART container will pass all necessary fields to your app via the `oauth_header` URL parameter. Specifically, the actual request sent to your app's backend server for index.html looks like this: 	
{% highlight html %}GET /index.html?oauth_header={Header value here...}
{% endhighlight  %}		

You need to first extract that header from the GET parameter: 
{% highlight html %}oauth_header = web.input().oauth_header
{% endhighlight  %}	

You'll also to need to URL-decode it:
{% highlight html %}oauth_header = urllib.unquote(oauth_header)
{% endhighlight  %}	

The field contains a complete OAuth Authorization header that includes a few extra fields, including notably smart\_record\_id, smart\_oauth\_token and smart\_oauth\_token\_secret. smart\_record\_id indicates the medical record ID of the current context, while the OAuth token and secret are the credentials your app needs to make REST API calls back into the SMART EMR. Why, then, are they themselves delivered in OAuth authorization header format? So you can verify that these tokens are authentic before you actually use them!

In other words, the SMART container is sending you credentials you can use to sign your API request. Those credentials are, themselves, signed, so you know they're okay to use.

Thankfully, you don't need to worry too much about the details, because we provide you with the utilities you need to extract the fields you need quickly and efficiently: 

{% highlight html %}
# parse it into a python dictionary
 oauth_params = oauth.parse_header(oauth_header)
{% endhighlight  %}	

Then get the specific parameters you need to sign your own API calls: 
{% highlight html %}
 record_id = oauth_params['smart_record_id']
 resource_credentials = {'oauth_token':        oauth_params['smart_oauth_token'],
                         'oauth_token_secret': oauth_params['smart_oauth_token_secret']}
{% endhighlight  %}	

The SMART Connect OAuth Header contains a few more parameters that will prove useful:

   <ul><li>
    smart_app_id: the identifier of the app being invoked</li>
    <li>smart_container_api_base: the base URL for API calls into the SMART EMR </li>
	</ul>

Now, why would your app need these, since presumably it knows its own ID and where the SMART container is located to make API, right?

In the simple case, yes, but in more advanced cases, your backend server might support multiple simultaneous apps made available to multiple SMART Containers. Thus, in a given setting, your app needs to know definitely which SMART Container it's connecting to and, if you're using one server to host multiple apps, which of your apps is being invoked in the first place. 

##Using SMART Client Libraries

Read up on the SMART Python Library.

Using those instructions, you can instantiate a SmartClient with the credentials obtained above:
{% highlight html %}
 client = SmartClient('my-app@apps.smartplatforms.org',
                      {'api_base' : 'http://sandbox-api.smartplatforms.org'},
                      {'consumer_key' : 'my-app@apps.smartplatforms.org',
                       'consumer_secret' : 'smartapp-secret'},
                      resource_credentials)
{% endhighlight  %}	

Since we're working against the SMART Reference EMR Sandbox, the APP_ID and consumer-level credentials are all fixed. Of course when connecting against a different SMART Container (e.g. your own), those will probably change. 

#Obtaining SMART Health Data

With our smart_client instance ready to go, loaded with the right credentials, we can start making API calls. Let's get the medication list:
{% highlight html %}
    medications = smart_client.records_X_medications_GET(record_id = record_id)
{% endhighlight  %}	
Just like in the first SMART App we built, the result is an SMARTResponse object containing an RDF graph of data, which we can query for just the fields we want:
{% highlight html %}
    query = """
        PREFIX dcterms:<http://purl.org/dc/terms/>
        PREFIX sp:<http://smartplatforms.org/terms#>
        PREFIX rdf:<http://www.w3.org/1999/02/22-rdf-syntax-ns#>
        SELECT  ?drugname ?rxcui
        WHERE {
           ?med rdf:type sp:Medication .
           ?med sp:drugName ?drugname_code .
           ?drugname_code dcterms:title ?drugname .
           ?drugname_code sp:code ?rxcui_url .
           ?rxcui_url dcterms:identifier ?rxcui .
        }
        """
 
    med_names_and_cuis = medications.graph.query(query)
    meds = [{'name': med[0], 'rxcui': med[1]} for med in med_names_and_cuis]
{% endhighlight  %}	

So far, we're not doing anything novel compared to our SMART Connect App. Let's make use of this fully capable backend we have, and integrate this data with other information. We'll use [RxNav](http://rxnav.nlm.nih.gov/), the National Library of Medicine's resource for RxNorm. In particular, we'll pull out the ingredients for each medication:
{% highlight html %}
    for med in meds:
      rxnav_info_xml = urllib.urlopen("http://rxnav.nlm.nih.gov/REST/rxcui/%s/related?rela=has_ingredient" % med['rxcui']).read()
      info = ElementTree.fromstring(rxnav_info_xml)
      med['ingredients'] = [ing.text for ing in info.findall('relatedGroup/conceptGroup/conceptProperties/name')]
{% endhighlight  %}	

Most of that code above is to quickly parse the XML we get back from RxNav and obtain the ingredient names.

Now that we've combined the SMART record data with a third-party data source, we can just render our HTML:

{% highlight html %}
    meds_html = "\n".join(["<li>%s<br /><small>ingredients: %s</small><br /><br /></li>" % (str(med['name']), ", ".join(med['ingredients'])) for med in meds])
{% endhighlight  %}	

and we're done.

\(Note that we're using clunky string concatenation to produce our HTML. Of course we recommend using a templating system, but we didn't want to burden the explanation with a templating language at this point.\)

#What Next?

<ul>
    <li>learn more about SMART REST API Calls with RxReminder.</li>
    <li>learn HOWTO Build SMART Background and Helper Apps so your App can take action while the user is offline. </li>   
</ul>