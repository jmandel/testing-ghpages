---
layout: page
title: Developers Documentation REST API
tagline: This is highly preliminary, not a commitment or final version of any particular API or data model. This is purely for internal collaboration and preview purposes.
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

The calls below are all written with respect to the base URL /. But any given SMART container will place all API calls its own base URL, e.g. [http://sample_smart_emr.com/smart-base/](http://sample_smart_emr.com/smart-base/).

Any individual item that can be retrieved via GET should have a fully-dereferenceable URI. To continue the example above, a medication in our sample EMR might have the URI [http://sample_smart_emr.com/smart-base/records/123456/medications/664373](http://sample_smart_emr.com/smart-base/records/123456/medications/664373). 

#Changelog

[Changes to API + Payloads](http://wiki.chip.org/smart-project/index.php?title=Developers_Documentation:_Changelog)

[Atom Feed](http://wiki.chip.org/smart-project/index.php?title=Developers_Documentation:_Changelog&action=history&feed=atom)

#Overview

The SMART API provides access to individual resources (medications, fulfillment events, prescription events, problems, etc.) and groups of these resources. 

##Read-only API 

Please note that for the time being, the SMART API remains read-only. We are excited about continuing to define our read/write API -- but we want make our early APIs as easy as possible for EMR and PHR vendors to expose. 

##REST Design Principles

In general you can interact with a:

* Group of resources using
        * GET to retrieve a group of resources such as /medications/
        * POST to add a group of resources such as /problems. POSTing will add new resources every time it is called; in other words, POST is not idempotent. 

* Single resource using
        * GET to retrieve a single resource such as /medications/{medication_id}
        * DELETE to remove a single resource
        * PUT to add a single resource tagged with an external_id. When a resource is PUT, it replaces any existing resource with the same external_id. In other words, PUT is idempotent. When PUTting a resource such as a medication that may contain child resources (e.g. fulfillment events), these child nodes must not be included in the graph. Rather, they must be separately attached with another API call once the parent medication is PUT and has received an internal SMART id. So, PUTting a medication with two fulfillments actually takes three API calls: one for the medication, and one for each (child) fulfillment event. 

#OWL Ontology File

The API calls listed below, as well as the RDF/XML payloads, are also defined in a machine-readable OWL file. The OWL file has been used to generate the documentation below, as well as our client-side REST libraries and API Playground app. 

#Container Calls

##App Manifest 

RDF Payload description

Returns a JSON SMART UI app manifest for the app matching {descriptor}, or 404. Note that {descriptor} can be an app ID like "got-statins@apps.smartplatforms.org" or an intent string like "view_medications".

 GET /apps/{descriptor}/manifest

Returns a JSON list of all SMART UI app manifests installed on the container.

 GET /apps/manifests/
 
##Capabilities

RDF Payload description

Get capabilities for a container


 GET /capabilities/


##Demographics

RDF Payload description

Get an RDF graph of sp:Demographics elements for all patients that match the query. Matching treats family_name and given_name as the *beginning* of a name. For instance given_name='J' matches /^J/i and thus matchs 'Josh'. Birthday is an ISO8601 string like "2008-03-21"; gender is "male" or "female". Gender, birthday, zipcode, and medical_record_number must match exactly.

{% highlight html %}
 GET /records/search?given_name={given_name}&family_name={family_name}&zipcode={zipcode}&birthday={birthday}&gender={gender}&medical_record_number={medical_record_number}

{% endhighlight  %}

##Ontology 

RDF Payload description

Get the ontology used by a SMART container

 GET /ontology
 
##User

RDF Payload description

Get users by name (or all users if blank)

 GET /users/search?given_name={given_name}&family_name={family_name}

Get a single user by internal ID

 GET /users/{user_id}

#Record Calls

##Alert

RDF Payload description
[edit] Allergy

RDF Payload description

Get all allergies for a patient

 GET /records/{record_id}/allergies/

Get allergies for a patient

 GET /records/{record_id}/allergies/{allergy_id}

##Demographics

RDF Payload description

Get all demographics for a patient

 GET /records/{record_id}/demographics

##Encounter

RDF Payload description

Get all encounters for a patient

 GET /records/{record_id}/encounters/

Get encounters for a patient

 GET /records/{record_id}/encounters/{encounter_id}

##Fulfillment

RDF Payload description

Get all fulfillments for a patient

 GET /records/{record_id}/fulfillments/

Get fulfillments for a patient

 GET /records/{record_id}/fulfillments/{fulfillment_id}

##Immunization

RDF Payload description

Get all immunizations for a patient

 GET /records/{record_id}/immunizations/

Get one immunization for a patient

 GET /records/{record_id}/immunizations/{immunization_id}

##Lab Result

RDF Payload description

Get all lab results for a patient

 GET /records/{record_id}/lab_results/

Get lab results for a patient

 GET /records/{record_id}/lab_results/{lab_result_id}

##Medical Record

RDF Payload description
##Medication

RDF Payload description

Get medication for a patient

 GET /records/{record_id}/medications/{medication_id}

Get all medications for a patient

 GET /records/{record_id}/medications/

##Problem

RDF Payload description

Get problems for a patient

 GET /records/{record_id}/problems/{problem_id}

Get all problems for a patient

 GET /records/{record_id}/problems/

##User Preferences

RDF Payload description

Get user preferences for an app

 GET /accounts/{user_id}/apps/{smart_app_id}/preferences

##VitalSigns

RDF Payload description

Get all vital signs for a patient

 GET /records/{record_id}/vital_signs/

Get vital signs for a patient

 GET /records/{record_id}/vital_signs/{vital_signs_id}




