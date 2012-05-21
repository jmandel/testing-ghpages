---
layout: page
title: Developers Documentation Changelog
tagline: for developers
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

#SMART Manifest

#API Updates since the $5K SMART Apps for Health Challenge

`JoshMandel` 15:11, 3 November 2011 (EDT) 

With feedback from the $5K SMART Apps for Health Challenge, we've been improving the SMART API to simplify the process of writing a SMART app. This includes some "breaking changes," meaning that existing apps will have to be updated (just a bit!). Here's a rundown of what's new

##Simpler app manifest

We simplified and flattened the SMART App manifest JSON structure. For more details, see our updated SMART manifest definition 

##App structure

SMART Apps no longer need to provide a bootstrap.html file. Instead, each app supplies a single index.html file. For more details, see our updated SMART Connect tutorial

## No more cookies for REST apps

When a SMART REST app launches, context is now provided via URL parameters rather than by cookie. For full details, see our updated SMART REST tutorial.

##Updated Data Model: Demographics

We're also introducing a more robust data model for demographics, including support for multiple telephone numbers and addresses via RDF vCard Ontology. For full details, see our Demographics Data Model.

##New Data Model: Vital Signs

We've added vital signs to the SMART data model, including height, weight, BMI, respiratory rate, heart rate, temperature, O2 saturation, and a detailed representation of blood pressures. For full details, see our Vital Signs Data Model.

## Updating your app

Here's a quick guide to making your app compatible with the changes described above.

##Update your manifest file

Update your app's manifest file, specifying your a name, description, author, id, version, mode scope, index, and icon.

##Remove bootstrap.html

With our new app structure, you don't need to supply this boilerplate file anymore.

##Wrap SMART JavaScript

Inside index.html, your app should include the JavaScript SMART API Client Library via:

{% highlight html %}
    <script src="http://sample-apps.smartplatforms.org/framework/smart/scripts/smart-api-client.js"></script>
{% endhighlight  %}

Once you've included this, you'll have access to an object called SMART. But before you use it, you'll need to make sure it's ready by calling SMART.ready(callback) with your code in the callback.

If needed, update your SMART client

If using the SMART Python (or Java) Client, download the latest version and update your local copy.
 If needed, pull REST OAuth tokens from URL parameter

If you're updating a SMART REST app , you'll need to grab context and OAuth tokens from a URL parameter called oauth_header, instead of looking for a cookie. If your app makes additional page requests to its back end (such as AJAX calls, redirects, page switching), you may have to explicitly pass the oauth header (which can be obtained in JavaScript from the SMART.credentials.oauth_header object) to the back end.

If needed, update your SPARQL queries

We're no longer using predicates from the "dublin core elements" namespace (http://purl.org/dc/elements/1.1/). Instead, all dublic core predicates now come from the "dublin core terms" namespace (http://purl.org/dc/terms/). So you should update any queries, e.g. from http://purl.org/dc/elements/1.1/date, you should update them to http://purl.org/dc/terms/date.

For full details about the new Demographics data model, see our Demographics Data Model. 

#Older Changes

Changed the URI for medications from to [http://rxnav.nlm.nih.gov/REST/rxcui/{rxcui}](http://rxnav.nlm.nih.gov/REST/rxcui/{rxcui}) . (Previously it had been [http://link.informatics.stonybrook.edu/rxnorm/RXCUI/{rxcui}](http://link.informatics.stonybrook.edu/rxnorm/RXCUI/{rxcui}). Also updated the sp:code property of CodedValue nodes, so this now points to a URI of type sp:Code (defining a coding system and identifier as needed -- please see payload examples).

`JoshMandel` 18:22, 2 March 2011 (EST) Changed the URI for medications from to: [http://rxnav.nlm.nih.gov/REST/rxcui/{rxcui}](http://rxnav.nlm.nih.gov/REST/rxcui/{rxcui}) . (Previously it had been [http://link.informatics.stonybrook.edu/rxnorm/RXCUI/{rxcui}](http://link.informatics.stonybrook.edu/rxnorm/RXCUI/{rxcui}). Also updated the sp:code property of CodedValue nodes, so this now points to a URI of type sp:Code (defining a coding system and identifier as needed -- please see payload examples).


`JoshMandel` 16:32, 21 February 2011 (EST) Added an AllergyException type, which is returned by GET /records/{rid}/allergies for a patient recorded as having no known allergies. Also added back very simple semantics for drug administration schedules, defining how much to take ("quantity") and how often ("frequency") in terms of [The Unified Code for Units of Measure](http://www.unitsofmeasure.org/). This is enough to describe thing like "1 pill three times daily," or "5 mL twice a day."


`JoshMandel` 18:14, 16 February 2011 (EST) Pared down Medication data model, eliminating strength, strengthUnit, dose, doseUnit, frequency, and route for the SMArt App Developers Challenge. (Our sample data provides RxNorm concept identifiers and free-text instructions, and we don't want to define fields that we're not going to populate. Note that apps can derive information about strength via RxNorm, including the RxNav REST service) 