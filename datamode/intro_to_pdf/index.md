---
layout: page
title: Developers Documentation Quick Introduction to RDF and SPARQL
tagline: third steps
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

#Developers Documentation Quick Introduction to RDF and SPARQL

The SMART API supplies patient record data in the form of an RDF graph. If you've never used (or even heard of!) RDF, this document should help you get up to speed. So let's jump right in! 

#What is RDF, anyway?

RDF, the Resource Description Framework, is a web standard "for representing information about resources" (this according to the [W3C's RDF Primer](http://www.w3.org/TR/2004/REC-rdf-primer-20040210/)). In brief, it's a flexible way to represent data in the form of sentences or "triples" that link a subject, a predicate, and an object. For example, let's say we want to represent the idea that "Mr. Smith takes atorvastatin". We might create the following triple

<ol><li>subject Mr. Smith</li>
    <li>predicate takes</li>
    <li>object atorvastatin </li>
	
There are two key ideas here

<ul><li>Everything (almost) is a resource.</li>
    <li>Resources are related by triples</li>
	</ul>

Let's explore each in more depth

## Everything (almost) is a resource