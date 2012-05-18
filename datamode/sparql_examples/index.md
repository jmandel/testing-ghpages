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

    <ul><li>the command line using [rasqal](http://librdf.org/rasqal/)</li>
    <li>Python, Perl, PHP, or Ruby using [librdf](http://librdf.org/)</li>
    <li>Java using [Jena](http://jena.sourceforge.net/), [Sesame](http://www.openrdf.org/), or [mulgara](http://www.mulgara.org/).</li>
    <li>Javascript When possible, we'll also provide examples that will work with the javascript library [rdfquery](http://code.google.com/p/rdfquery/), which is automatically provided by the [smart-api-client.js](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_smart-api-client.js_Reference).</li>
	</ul>
	
#The Examples	

##Examining a Medication List

Let's work with some medication data, the kind returned by a SMART API call to GET /records/{record_id}/medications/. To start off, let's write a query to find the name of each medication in the list: 	 