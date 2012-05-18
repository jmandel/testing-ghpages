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
    <li>object\ atorvastatin </li>

There are two key ideas here

<ul><li>Everything (almost) is a resource.</li>
    <li>Resources are related by triples</li>
	</ul>





##Resources are related by triples

In RDF, the only way to represent relations among resources is by creating triples. If graph theory is your thing, you can think of triples as arcs in a directed graph from subject to predicate to object. The same resource can be the subject (or object) or multiple triples. For example, consider my SMART haiku. In addition to the triple above, I could some more triples

<ol><li>subject [http://dilute.net/poems/25](http://dilute.net/poems/25)</li>
    <li>predicate dc:creator (Dublin Core vocabulary's 'creator' predicate)</li>
    <li>object [http://joshuamandel.com/me](http://joshuamandel.com/me)</li>
	</ol>


<ol><li>subject [http://joshuamandel.com/me](http://joshuamandel.com/me)</li>
    <li>predicate foaf:name (FOAF vocabulary's 'name' predicate)</li>
    <li>object "Josh Mandel"</li>
	</ol>

Note that I am the object of one triple (as the creator of the haiku) and the subject of another (as a person with a name)!

What about more complex relationships? For example, what if I want to represent the fact that my breakfast this morning consisted of Joe's O's, milk, and coffee? This is an open-ended data-modeling exercise, but I'll just point out one approach which involves creaing a resource for "the stuff I had for breakfast this morning", and adding relations to that. So then (in sketch form) we'd have

<ol><li>subject http://joshuamandel.com/me</li>
    <li>predicate http://joshuamandel.com/my_food_vocabulary/ate</li>
    <li>object: _stuff_I_ate_this_morning </li>
	</ol>


<ol><li>subject \_stuff\_I\_ate\_this\_morning</li>
    <li>predicate rdfli (RDF vocabulary's 'list item' predicate)</li>
    <li>object "Joe's O's"</li>
	</ol>


<ol><li>subject \_stuff\_I\_ate\_this_morning\</tt>
    <li>predicate rdf\li </li>
    <li>object "milk"</li>
	</ol>


<ol><li>subject \<tt>\_stuff\_I\_ate\_this\_morning </li>
    <li>predicate rdf\li </li>
    <li>object "coffee"</li> 

Notice that I've loosely referred to a resource here as the "bunch of stuff I ate this morning". I didn't give it a formal URI, because it doesn't exist outside of the context of this particular RDF graph, and it's entirely defined by its relations above. For cases like this, RDF provides anonymous or blank nodes whose identifiers have meaning only within the context of a particular graph. 



##Representing RDF Graphs

So far, we've been talking about RDF graphs as theoretical sets of triples. How do we write down or "serialize" an RDF graph in a way that lets us share it with others? There are in fact several standard notations for representing an RDF graph. The simplest representation is to write triples out, one per line, with a period at the end of each line. URIs are enclosed in angle brackets (e.g. \<[http://my_uri](http://my_uri)>\; blank nodes are prefaced with the \_ prefix (e.g. \_my-blank-node), and strings are enclosed in quotes (e.g. "my string value")

<http://dilute.net/poems/25> <http://purl.org/dc/terms/title> \"Haiku entitled /  Substitutability: / the SMART way to go.\" .

An XML-based representation known as RDF/XML serializes the same triple more verbosely
{% highlight html %}
<?xml version="1.0"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#" xmlns:terms="http://purl.org/dc/terms/">
	<rdf:Description rdf:about="http://dilute.net/poems/25">
		<terms:title>Haiku entitled /  Substitutability: / the SMART way to go.</terms:title>
	</rdf:Description>
</rdf:RDF>
{% endhighlight  %}



#And what about SPARQL?

SPARQL is a query language for interacting with RDF graphs. The syntax is designed to look a bit like SQL, the structured query language used with relational databases. The W3C maintains an [extremely readable](http://www.w3.org/TR/rdf-sparql-query/) standard that's peppered with examples. Here, we'll not even skim the surface... 




##A simple SPARQL query

Given our breakfast graph above, let's write a query to find all the things I ate! Here's a first attempt (not quite perfect) 

{% highlight html %}
PREFIX food: <http://joshuamandel.com/my_food_vocabulary/> 
SELECT ?f WHERE
{
  <http://joshuamandel.com/me> food:ate ?f.
} 
{% endhighlight  %}


A bit of syntax I've defined a prefix called "food" which I'll use to refer to my personal food vocabualry. This is just for readability; it lets me later write foodate instead of the more verbose [http://joshuamandel.com/my_food_vocabulary/ate](http://joshuamandel.com/my_food_vocabulary/ate).

Now here's what the query does: it looks for triples that match the pattern inside the WHERE clause. In this case, triples whose subject is me; whose predicate is food:ate and whose object can be anything (indicated by the question mark in ?f). My decision to use ?f as a variable name was completely discretionary. I could have called it ?nourishment or ?xyzzy. The name only matters within the context of my query.

But this query has a problem it returns the blank node \_stuff\_I\_ate\_this\_morning -- and not the actual foods! Let's fix it by adding to our WHERE clause

{% highlight html %}
PREFIX food: <http://joshuamandel.com/my_food_vocabulary/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> 
SELECT ?individual_food WHERE
{
  <http://joshuamandel.com/me> food:ate ?bunch_of_food.
  ?bunch_of_food rdf:li ?individual_food.
} 
{% endhighlight  %}

Now our where clause includes two statements we're looking for individual foods that are items in the list of foods eaten by me. In other words, now we're drilling down into the bunch of food to pull out individual items! This returns a list of three bindings for the ?individual_food "coffee", "milk", and "Joe's O's".

This was just the briefest introduction to the anatomy of a SPARQL query. For lots more specific examples, try SPARQL examples for SMART. 