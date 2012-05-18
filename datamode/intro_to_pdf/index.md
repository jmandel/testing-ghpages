---
layout: page
title: Developers Documentation Quick Introduction to RDF and SPARQL
tagline: for developers
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

The SMART API supplies patient record data in the form of an RDF graph. If you've never used (or even heard of!) RDF, this document should help you get up to speed. So let's jump right in! 

#What is RDF, anyway?

RDF, the Resource Description Framework, is a web standard "for representing information about resources" (this according to the [W3C's RDF Primer](http://www.w3.org/TR/2004/REC-rdf-primer-20040210/)). In brief, it's a flexible way to represent data in the form of sentences or "triples" that link a subject, a predicate, and an object. For example, let's say we want to represent the idea that "Mr. Smith takes atorvastatin". We might create the following triple

<ol><li>subject Mr. Smith</li>
    <li>predicate takes</li>
    <li>object atorvastatin </li>
	</ol>
	
There are two key ideas here

<ul><li>Everything (almost) is a resource.</li>
    <li>Resources are related by triples</li>
</ul>

Let's explore each in more depth

## Everything (almost) is a resource

In RDF, every triple has a resource as its subject. In our example, we call Mr. Smith a "resource" because he is a particular guy out there in the world. He is not just the string of letters "M-r-.-S-m-i-t-h." Importantly, if I know Mr. Alex Smith and you know Mr. Bob Smith, we are not talking about the same resource! To prevent these kinds of mix-ups, resources in RDF aren't just identified by strings like "Mr. Smith." Instead, they're represented by Uniform Resource Identifiers basically URLs that provide a built-in namespace. For example, let's say my Mr. Smith maintains a web site at [http://alexsmith.somedomain.com](http://alexsmith.somedomain.com). I might refer to him by the URL [http://alexsmith.somedomain.com/me](http://alexsmith.somedomain.com/me]). Now you certainly wouldn't confuse my Mr. Smith for yours! (Note there doesn't have to be an actual web page served at the address of a URI. The important thing is that the URI identifies a resource. Uniformly.) 

What about the predicate in our example, the word "takes"? Predicates in RDF are triples, too. If we just used the string "takes" as our predicate, again we might mean different things I might mean "consumes a drug, as part of a daily regimen", and you (cynic!) might mean "steals from his wife's pillbox on Thursday mornings." To resolve this ambiguity, I could represent 'takes' as [http://joshuamandel.com/my_drug_vocabulary/takes](http://joshuamandel.com/my_drug_vocabulary/takes). Over time, I could build up a rich vocabulary with all kinds of terms, and use these as predicates in my RDF triples. In general, things work best when people can agree on the meanings of terms and use a shared vocabulary. So folks build up publically defined vocabularies such as [FOAF](http://xmlns.com/foaf/spec/) (used to describe the elements in social networks like friends, names, and birthdays) or [Dublic Core](http://purl.org/dc/elements/1.1/) (used to describe metadata like the Titles, Creators, and Publishers or resources). These shared vocabularies become the basis for rich representation (and interpretation) of information about resources. 

And finally, what about "atorvastatin"? Again, the best way to represent a concept like atorvastatin is as a URI that everyone can agree on. One possibility is to use the drug's RxNorm Concept ID (in this case, 83367) as part of the URI. For example, SMART uses the URI [http://link.informatics.stonybrook.edu/rxnorm/RXCUI/83367](http://link.informatics.stonybrook.edu/rxnorm/RXCUI/83367), sharing a vocabulary with Stonybrook. But recall we said almost everything is a resource. If we want, RDF lets us use a simple string as the object of a triple. So, for example, consider this representation of a Haiku

<ol><li>subject [http://dilute.net/poems/25](http://dilute.net/poems/25)</li>
    <li>predicate dcterms title (Dublin Core Terms vocabulary's 'title' predicate)</li>
    <li>object "Haiku entitled Substitutability the SMART way to go." </li>

In this case, I don't need to point to a resource as the title of my haiku. The title is really just a string, after all -- so I can just represent it as such. 

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

<ul><li>subject http://joshuamandel.com/me</li>
    <li>predicate http://joshuamandel.com/my_food_vocabulary/ate</li>
    <li>object _stuff_I_ate_this_morning </li>
	</ul>


<ul><li>subject _stuff_I_ate_this_morning</li>
    <li>predicate rdfli (RDF vocabulary's 'list item' predicate)</li>
    <li>object "Joe's O's"</li>
	</ul>


<ul><li>subject _stuff_I_ate_this_morning\</tt>
    <li>predicate rdf li </li>
    <li>object "milk"</li>
	</ul>


<ul><li>subject <tt>_stuff_I_ate_this_morning </li>
    <li>predicate rdf li</li>
    <li>object "coffee"</li> 
	</ul>

Notice that I've loosely referred to a resource here as the "bunch of stuff I ate this morning". I didn't give it a formal URI, because it doesn't exist outside of the context of this particular RDF graph, and it's entirely defined by its relations above. For cases like this, RDF provides anonymous or blank nodes whose identifiers have meaning only within the context of a particular graph. 
