---
layout: page
title: Developers Documentation SPARQL Examples for SMART
tagline: for developers
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

#SPARQL by Example

This page provides a set of example queries to help you get started interacting with SMART patient record data. We'll build out several example SPARQL queries, but please feel free to add on new material as you discover useful tidbits! 

#Running Queries- Live in-browser, or in your own environment

You can use the form below to try out queries right away. Please note that to use the live query tool, you'll need to include a FROM <graph> clause in your query to supply data. If you're running these queries in your own environment, you'll run them in the context of a particular graph (e.g. Patient Smith's medication graph), rather than specifying FROM directly in the query.

Or you could run these examples on your own, via

* the command line using [rasqal](http://librdf.org/rasqal)
* Python, Perl, PHP, or Ruby using [librdf](http://librdf.org)
* Java using [Jena](http://jena.sourceforge.net), [Sesame](http://www.openrdf.org), or [mulgara](http://www.mulgara.org).
* Javascript When possible, we'll also provide examples that will work with the javascript library [rdfquery](http://code.google.com/p/rdfquery/), which is automatically provided by the [smart-api-client.js](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_smart-api-client.js_Reference).
	
	
#The Examples	

##Examining a Medication List

Let's work with some medication data, the kind returned by a SMART API call to GET /records/{record_id}/medications/. To start off, let's write a query to find the name of each medication in the list: 	 

Note the use of the "distinct" keyword: as in SQL, distinct will prune the list for duplicates. So if the two medications in the patient record have the same name, this query will collapse them into one result.

With rdfquery, we can achieve a similar result

{% highlight javascript %}
SMART.MEDS_get().success(function(response) {
     var fill_dates = response.graph
                           .where("?m rdf:type sp:Medication")
                           .where("?med sp:drugName ?medc")
                           .where("?medc dcterms:title ?t");
  });
{% endhighlight  %}


{% highlight javascript %}
	Find Medication Quantities + Frequencies
	
	PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
	PREFIX sp: <http://smartplatforms.org/terms#>
	PREFIX dcterms: <http://purl.org/dc/terms/>
	
	SELECT  ?t ?quant_val ?quant_unit ?freq_val ?freq_unit
	  FROM <http://sandbox-api.smartplatforms.org/records/2169591/medications/>
	WHERE {
	  ?m rdf:type sp:Medication .
	  ?m sp:drugName ?medc.
	  ?medc dcterms:title ?t.
	  ?m sp:quantity ?q.
	  ?q sp:value ?quant_val.
	  ?q sp:unit ?quant_unit.
	  ?m sp:frequency ?f.
	  ?f sp:value ?freq_val.
	  ?f sp:unit ?freq_unit.
	}
{% endhighlight  %}

Note the use of the "distinct" keyword: as in SQL, distinct will prune the list for duplicates. So if the two medications in the patient record have the same name, this query will collapse them into one result. 

Find Medication Fulfillment Dates

{% highlight javascript %}
	PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
	PREFIX sp: <http://smartplatforms.org/terms#>
	PREFIX dcterms: <http://purl.org/dc/terms/>
	SELECT  distinct ?t ?fill_date
	  FROM <http://sandbox-api.smartplatforms.org/records/2169591/medications/>
	WHERE {
	  ?m rdf:type sp:Medication .
	  ?m sp:drugName ?medc.
	  ?medc dcterms:title ?t.
	  ?m sp:fulfillment ?fill.
	  ?f dcterms:date ?fill_date.
	}
{% endhighlight  %}

With rdfquery, we can achieve a similar result

{% highlight javascript %}
 SMART.MEDS_get().success(function(response) {
     var fill_dates = response.graph
                           .where("?m rdf:type sp:Medication")
                           .where("?m sp:fulfillment ?fill")
                           .where("?med sp:drugName ?medc")
                           .where("?medc dcterms:title ?t")
                           .where("?fill dcterms:date ?fill_date");
   });
   
{% endhighlight  %}

Find Medications Fulfilled Since January 2009

{% highlight javascript %}
	PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
	PREFIX sp: <http://smartplatforms.org/terms#>
	PREFIX dcterms: <http://purl.org/dc/terms/>
	SELECT  distinct ?t
	  FROM <http://sandbox-api.smartplatforms.org/records/2169591/medications/>
	WHERE {
	  ?m rdf:type sp:Medication .
	  ?m sp:drugName ?medc.
	  ?medc dcterms:title ?t.
	  ?m sp:fulfillment ?fill.
	  ?f dcterms:date ?fill_date.
	  FILTER( xsd:dateTime(?fill_date) > "2009-01-01T00:00:00Z"^^xsd:dateTime )
	} ORDER BY (?t)

{% endhighlight  %}

Again a similar result with rdfquery

{% highlight javascript %}
	SMART.MEDS_get().success(function(response) {
		 var fill_dates = response.graph
							   .where("?m rdf:type sp:Medication")
							   .where("?m sp:fulfillment ?fill")
							   .where("?med sp:drugName ?medc")
							   .where("?medc dcterms:title ?t")
							   .where("?fill dcterms:date ?fill_date")
							   .filter(function() {
								  return Date.parse(this.fill_date.value) > Date.parse("2009-01-01T00:00:00Z");
								});
	   });
	   
{% endhighlight  %}

##Getting some Demographics

Here's a query that pulls out the first and last name from a patient's demographics

##Find Patient's Name

{% highlight javascript %}
	PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
	PREFIX sp: <http://smartplatforms.org/terms#>
	PREFIX dcterms: <http://purl.org/dc/terms/>
	PREFIX foaf:<http://xmlns.com/foaf/0.1/>
	
	SELECT ?d ?fn ?ln
	  FROM <http://sandbox-api.smartplatforms.org/records/2169591/demographics>
	
	WHERE {
	  ?d rdf:type foaf:Person.
	  ?d foaf:givenName ?fn.
	  ?d foaf:familyName ?ln.
	}
{% endhighlight  %}

With rdfquery, we can achieve a similar result: 

{% highlight javascript %}
	SMART.DEMOGRAPHICS_get().success(function(response) {
		 var person = response.graph
							   .where("?d rdf:type <http://xmlns.com/foaf/0.1/Person>")
							   .where("?d <http://xmlns.com/foaf/0.1/givenName> ?fn")
							   .where("?d <http://xmlns.com/foaf/0.1/familyName> ?ln");
	  });
{% endhighlight  %}

##Getting some Problems

Here's a query that pulls out the name of each problem

##Find Patient's Problems
{% highlight javascript %}
	PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
	PREFIX sp: <http://smartplatforms.org/terms#>
	PREFIX dcterms: <http://purl.org/dc/terms/>
	
	SELECT ?p
	 FROM <http://sandbox-api.smartplatforms.org/records/2169591/problems/>
	WHERE {
	 ?pr rdf:type sp:Problem .
	 ?pr sp:problemName ?pn .
	 ?pn dcterms:title ?p .
	}
{% endhighlight  %}


With rdfquery, we can achieve a similar result

{% highlight javascript %}
SMART.PROBLEMS_get().success(function(response) {
     var person = response.graph
                           .where("?pr rdf:type sp:Prblem")
                           .where("?pr sp:problemName ?pn")
                           .where("?pn dcterms:title ?p");
  });
{% endhighlight  %}


##Getting some Problems

Here's a query that pulls out the name of each problem

##Finding Quantitative Labs

{% highlight javascript %}
	PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
	PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
	PREFIX sp: <http://smartplatforms.org/terms#> 
	PREFIX dcterms: <http://purl.org/dc/terms/> 
	PREFIX foaf:<http://xmlns.com/foaf/0.1/> 
	PREFIX v:<http://www.w3.org/2006/vcard/ns#> 
	SELECT  DISTINCT ?loinc_title ?lab_title ?value ?unit ?minvalue ?minunit ?maxvalue ?maxunit ?ncrminvalue ?ncrminunit
	 FROM <http://sandbox-api.smartplatforms.org/records/2169591/lab_results/> 
	WHERE { 
	  ?lr rdf:type sp:LabResult. 
	  ?lr sp:labName ?labName . 
	  ?labName dcterms:title ?loinc_title . 
	  OPTIONAL{?labName sp:codeProvenance ?codeProvenance .} 
	  OPTIONAL{?codeProvenance dcterms:title ?lab_title.} 
	  ?lr sp:quantitativeResult ?quantitativeResult . 
	  ?quantitativeResult sp:valueAndUnit ?valueAndUnit . 
	  ?valueAndUnit sp:value ?value . 
	  ?valueAndUnit sp:unit ?unit . 
	  ?quantitativeResult sp:normalRange ?normalRange . 
	  ?normalRange sp:minimum ?minimum . 
	  ?minimum sp:value ?minvalue . 
	  ?minimum sp:unit ?minunit . 
	  ?normalRange sp:maximum ?maximum . 
	  ?maximum sp:value ?maxvalue . 
	  ?maximum sp:unit ?maxunit . 
	  OPTIONAL{?quantitativeResult sp:nonCriticalRange ?nonCriticalRange .} 
	  OPTIONAL{?nonCriticalRange sp:minimum ?ncrminimum .} 
	  OPTIONAL{?ncrminimum sp:value ?ncrminvalue .} 
	  OPTIONAL{?ncrminimum sp:unit ?ncrminunit .} 
	  OPTIONAL{?lr sp:status ?status .} 
	  OPTIONAL{?status dcterms:title ?status_title .} 
	  OPTIONAL{?lr sp:abnormalInterpretation ?abnormalInterpretation.} 
	  }
{% endhighlight  %}

