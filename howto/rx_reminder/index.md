---
layout: page
title: RxReminder App
tagline: for developer
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>


#Developers Documentation: RxReminder - SMART REST in Action

You should read [HOWTO Build a SMART App - REST API](../build_a_rest_app) Calls first. 

#Choosing the right tools

Since you can write a SMART REST app in any language using any toolkit, there's a lot of flexibility. In general, you'll want to look for a language with existing OAuth libraries to handle the details of signing requests to the SMART container. Here we'll illustrate the highlights with a simple python-based SMART REST app called *RxReminder. RxReminder* is written using the web.py HTTP Framework, but we won't delve into [web.py](http://webpy.org/) here. All you need to understand is that web.py provides a simple mechanism to map python functions to URLs. 

Code snippets are provided below, but please see [the full source tree on github](http://github.com/chb/smart_rx_reminder). 

#App Structure

First, recall that every SMART UI app must serve: 
<ol><li>
index.html: loads smart-api-client.js to provide the app's functionality. 
</li>
</ol>

Just like the app in the HOWTO, we serve the index file as follows in webpy: 

{% highlight javascript %}
 urls = ('/smartapp/index.html',     'RxReminder')
 
 class RxReminder:
    def GET(self):
        # good stuff coming here
		
{% endhighlight  %}

#Initializing the SMART API Client

Now we're getting to the fun part: handling requests for index.html. We'll be using the Python SMART API client libraries to interact with the SMART container. For authentication, we'll use the OAuth token and secret provided in a URL parameter that came along with the request for our index.html, just like in the HOWTO. 

{% highlight javascript %}
 smart_oauth_header = web.input().oauth_header
     smart_oauth_header = urllib.unquote(smart_oauth_header)
     
     # Pull out OAuth params from the header
     oa_params = oauth.parse_header(smart_oauth_header)
     resource_tokens={'oauth_token':       oa_params['smart_oauth_token'],
                      'oauth_token_secret':oa_params['smart_oauth_token_secret']}

     # instantiate the smart client
     client = SmartClient(SMART_SERVER_OAUTH['consumer_key'], 
                          SMART_SERVER_PARAMS, 
                          SMART_SERVER_OAUTH, 
                          resource_tokens)
{% endhighlight  %}

If you're building your application from scratch without using the sample code, you'll want to define SMART\_SERVER\_OAUTH and SMART\_SERVER\_PARAMS: 

{% highlight javascript %}

# Basic configuration:  the consumer key and secret we'll use
 # to OAuth-sign requests.
 SMART_SERVER_OAUTH = {'consumer_key': 'my-app@apps.smartplatforms.org', 
                       'consumer_secret': 'smartapp-secret'}
 
 
 # The SMART contianer we're planning to talk to
 SMART_SERVER_PARAMS = {
     'api_base' : 'http://sandbox-api.smartplatforms.org'
 }
 
{% endhighlight  %}

#Generating Reminders

Now that we've got access tokens, and initialized a client object, let's interact with some health data! *RxReminder* needs to get a list of medications due for refills. We'll accomplish this by finding the most recent fulfillment event for each medication. Then we'll look at how many days' supply of medication were dispensed to find out on what day the patient will run out.

We use our SmartClient object called client to fetch a SMART Response object including a list of medications, via `records_X_medications_GET() (or, in more verbose form, client.get("/records/%s/medications/{record_id}")` SMART API call.

Then we'll get fancy with RDF, running a SPARQL query to find a list of fills for each medication in meds.graph. Finally, we'll loop through the fills to find the most recent one, using the dispensed quantity to determine when the patient will run out of medication. 


{% highlight javascript %}

	# Represent the list as an RDF graph
			meds = client.records_X_medications_GET()
			
			# Find a list of all fulfillments for each med.
			q = """
				PREFIX dc:<http://purl.org/dc/elements/1.1/>
				PREFIX dcterms:<http://purl.org/dc/terms/>
				PREFIX sp:<http://smartplatforms.org/terms#>
				PREFIX rdf:<http://www.w3.org/1999/02/22-rdf-syntax-ns#>
				   SELECT  ?med ?name ?quant ?when
				   WHERE {
						  ?med rdf:type sp:Medication .
						 ?med sp:drugName ?medc.
						  ?medc dcterms:title ?name.
						  ?med sp:fulfillment ?fill.
						  ?fill sp:dispenseDaysSupply ?quant.
						  ?fill dc:date ?when.
				   }
				"""
			pills = RDF.SPARQLQuery(q).execute(meds.graph)
	
			# Find the last fulfillment date for each medication
			self.last_pill_dates = {}
	
			for pill in pills:
				self.update_pill_dates(*pill)
	
			#Print a formatted list
			return header + self.format_last_dates() + footer

{% endhighlight  %}

The details of finding the most recent prescription are in the update_pill_dates function. Please [check out the source on github](http://github.com/chb/smart_rx_reminder) to see the whole picture! 