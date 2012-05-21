---
layout: page
title: Developers Documentation SMART Data Model
tagline: This is highly preliminary, not a commitment or final version of any particular API or data model. This is purely for internal collaboration and preview purposes.
includenav: smartnav.markdown
---
{% include JB/setup %}

<div id="toc"> </div>

#Changelog

[Changes to API + Payloads](http://wiki.chip.org/smart-project/index.php?title=Developers_Documentation:_Changelog)

[Atom Feed](http://wiki.chip.org/smart-project/index.php?title=Developers_Documentation:_Changelog&action=history&feed=atom)

#RDF Overview

SMART API calls return data in the form of RDF graphs. Within these graphs, top-level SMART data objects have fully dereferenceable URIs. For instance, let's consider a SMART-enabled EMR hosted with its API base at [http://sample_smart_emr.com/smart-app/](http://sample_smart_emr.com/smart-app/). An individual medication in this EMR might be retrievable via

GET [http://sample_smart_emr.com/smart-app/records/123456/medications/664373](http://sample_smart_emr.com/smart-app/records/123456/medications/664373).

When you issue a GET request for a top-level SMART object like a medication, you'll receive a graph containing

A) The object itself
B) All properties linking it to "core data" elements (such as a medication's drugName, which is a CodedValue node)
C) All properties linking it to other top-level SMART objects
D) All "core data" elements belonging to the elements in (C)

So, for instance, when you GET a medication, you'll get all the information about that medication and any fulfillments that belong to it! Likewise, when you GET a fulfillment, you'll get all the information about that fulfillment and the medication to which it belongs. 

Another way to say this is: when you make an API call for data, the graph you receive goes one top-level SMART element deep. 

#Important note: RDF/XML Serializations are not unique!

It's important to understand that SMART API calls return RDF graphs [serialized as RDF/XML](http://www.w3.org/TR/REC-rdf-syntax/) -- and for a given graph, there are multiple possible serializations. We don't make guarantees about how a graph will be serialized, beyond saying that it's valid RDF/XML. So to consume the SMART API, you should parse the RDF/XML payloads as RDF/XML -- don't try to query them directly with xpath, for example! 

To give a concrete example, a medication might be serialized as

{% highlight html %}
<sp:Medication>
  ... { additional properties here }....
</sp:Medication>
{% endhighlight  %}

Or you could see the equivalent

{% highlight html %}
<rdf:Description>
      <rdf:type rdf:resource="http://smartplatforms.org/terms#Medication"/>
  ... { additional properties here }....
</rdf:Description>
{% endhighlight  %}

#Namespaces

A common set of namespace prefixes is used for the examples below. These are


<table width="100%" border="0" cellspacing="0" class="table table-striped">
  <tr>
    <td>dcterms</td>
    <td><a href="http://purl.org/dc/terms/">http://purl.org/dc/terms/</a></td>
    <td>Dublin core terms</td>
  </tr>
  
   <tr>
    <td>foaf</td>
    <td><a href="http://xmlns.com/foaf/0.1/">http://xmlns.com/foaf/0.1/</a></td>
    <td>Friend of a friend</td>
  </tr>
  
   <tr>
    <td>rdf</td>
    <td><a href="http://www.w3.org/1999/02/22-rdf-syntax-ns#">http://www.w3.org/1999/02/22-rdf-syntax-ns#</a></td>
    <td>Resource description framework</td>
  </tr>
  
  <tr>
    <td>sp</td>
    <td><a href="http://smartplatforms.org/terms#">http://smartplatforms.org/terms#</a></td>
    <td>Smart platforms root namespace</td>
  </tr>
  
  <tr>
    <td>v</td>
    <td><a href="http://www.w3.org/2006/vcard/ns#">http://www.w3.org/2006/vcard/ns#</a></td>
    <td>vCard namespace</td>
  </tr>
  
</table>

RDF Namespaces

#Naming conventions in SMART Ontology

We use the prefix sp to designate the http://smartplatforms.org/terms# namespace. Within this namespace, we use a simple convention to differentiate between classes and predicates

* Class names are designated by CamelCase with a capitalized first character. Examples- sp Medication, sp LabResult. 

* Predicate names are designated by camelCase with a lower-case first character. Examples- sp medication or sp valueAndUnit. 


You may wonder why classes and predicates tend to have similar names. For example, we define a class sp:Medication as well as a predicate sp:medication. Here's why: a predicate like sp:medication is used to indicate that a clinical statement is associated with a medication; a class like sp:Medication is used to indicate that a clinical statement is a medication. For example, each sp:Fulfillment statement is associated with a sp:Medication statement via the predicate sp:medication. This makes sense when we consider a few RDF triples that expresses the basic pattern, associating a fulfillment with its medication via the sp:medication predicate

{% highlight html %}

_:f123 rdf:type sp:Fulfillment.   # declares a Fulfillment statement
_:m456 rdf:type sp:Medication.    # declares a Medication statement

_f123 sp:medication _:m456.       # links the Fulfillment to its Medication

{% endhighlight  %}

#Clinical Statement Types

##Alert RDF

Alert is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)

Alerts are a way for an application to generate a message (about a patient record) intended for a human recipient. 

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"> 

  <sp:Alert> 
    <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
    <sp:notes>Patient with T2DM is overdue for HbA1c</sp:notes>
    <sp:severity>
      <sp:CodedValue>
        <dcterms:title>Warning</dcterms:title>
        <sp:code>
          <spcode:AlertSeverity rdf:about="http://smartplatforms.org/terms/codes/AlertSeverity#warning">
            <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
            <dcterms:title>Warning</dcterms:title> 
            <sp:system>http://smartplatforms.org/terms/codes/AlertSeverity#</sp:system>
            <dcterms:identifier>warning</dcterms:identifier> 
          </spcode:AlertSeverity>   
        </sp:code>
      </sp:CodedValue>

    </sp:severity> 
  </sp:Alert>
</rdf:RDF>
{% endhighlight  %}



<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Alert" class="external free" title="http://smartplatforms.org/terms#Alert" rel="nofollow">http://smartplatforms.org/terms#Alert</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>alertLevel</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Alert" class="external text" title="http://smartplatforms.org/terms#Alert" rel="nofollow">sp:Alert</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#AlertLevel_code_RDF" title=""> AlertLevel</a>
</td></tr>
<tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Alert" class="external text" title="http://smartplatforms.org/terms#Alert" rel="nofollow">sp:Alert</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%">notes<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Alert" class="external text" title="http://smartplatforms.org/terms#Alert" rel="nofollow">sp:Alert</a>]</small>
</td><td width="50%">Message intended for a human recipient.	 [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>


##Allergy RDF

Allergy is a subtype of and inherits properties from: SMART Statement


SMART provides structure for representing alleriges accoring to well-specified standard vocabularies. A patient with no known allergies produces an AllergyExclusion providing further details (see below). Otherwise, each allergy has a category (drug, food, etc.), a severity, a reaction, and a substance or class of substances. Since we want these allergies to work as inputs to automated computations, we require specific coding systems to represent substances and classes of substances.

1. A drug allergy to a particular drug (e.g. cephalexin) must define a "substance" field with an ingredient-type RxNorm code (tty="IN"). (This is to avoid overly-specific statements like "the patient is allergic to a 500mg cephalexin oral tablet"!)

2. A drug allergy to an entire class of drugs (e.g. sulfonamides) must define a "class" field with an NDFRT code for the drug class.

3. A food or environmental allergy must define a "substance" field with a UNII code.

For instance, below are two allergies: first, an allergy to the entire class of sulfonamides (note the sp:class predicate and the NDFRT code provided); then, an allergy to a single cephalosporin drug, cephalexin (note the sp:substance predicate and the RxNorm Ingredient CUI provided)



{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"> 
      <sp:Allergy>
      <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
      <sp:drugClassAllergen>
      <sp:CodedValue>
        <dcterms:title>Sulfonamide Antibacterial</dcterms:title>
        <sp:code>
        <spcode:NDFRT rdf:about="http://purl.bioontology.org/ontology/NDFRT/N0000175503">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Sulfonamide Antibacterial</dcterms:title>
          <sp:system>http://purl.bioontology.org/ontology/NDFRT/</sp:system>
          <dcterms:identifier>N0000175503</dcterms:identifier>
        </spcode:NDFRT>
        </sp:code>
      </sp:CodedValue>

      </sp:drugClassAllergen>
      <sp:severity>
      <sp:CodedValue>
          <dcterms:title>Severe</dcterms:title>
        <sp:code>
        <spcode:AllergySeverity rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/24484000">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Severe</dcterms:title>
          <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
          <dcterms:identifier>24484000</dcterms:identifier>
        </spcode:AllergySeverity>
        </sp:code>
      </sp:CodedValue>
      </sp:severity>
      <sp:allergicReaction>
      <sp:CodedValue>
          <dcterms:title>Anaphylaxis</dcterms:title>
        <sp:code>
        <spcode:SNOMED rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/39579001">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Anaphylaxis</dcterms:title>
          <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
          <dcterms:identifier>39579001</dcterms:identifier>
        </spcode:SNOMED>
        </sp:code>
      </sp:CodedValue>

      </sp:allergicReaction>
      <sp:category>
      <sp:CodedValue>
          <dcterms:title>Drug allergy</dcterms:title>   
        <sp:code>
        <spcode:AllergyCategory rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/416098002">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Drug allergy</dcterms:title>   
          <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
          <dcterms:identifier>416098002</dcterms:identifier>
        </spcode:AllergyCategory>
        </sp:code>
      </sp:CodedValue>

      </sp:category>
   </sp:Allergy>

  <sp:Allergy>
      <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
      <sp:drugAllergen>
      <sp:CodedValue>
          <dcterms:title>Cephalexin</dcterms:title>
        <sp:code>
        <spcode:RxNorm_Ingredient rdf:about="http://purl.bioontology.org/ontology/RXNORM/2231">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Cephalexin</dcterms:title>
          <sp:system>http://purl.bioontology.org/ontology/RXNORM/</sp:system>
          <dcterms:identifier>2231</dcterms:identifier>
        </spcode:RxNorm_Ingredient>
        </sp:code>
      </sp:CodedValue>

      </sp:drugAllergen>
      <sp:severity>
      <sp:CodedValue>
          <dcterms:title>Severe</dcterms:title>
        <sp:code>
        <spcode:AllergySeverity rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/24484000">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Severe</dcterms:title>
          <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
          <dcterms:identifier>24484000</dcterms:identifier>
        </spcode:AllergySeverity>
        </sp:code>
      </sp:CodedValue>

      </sp:severity>
      <sp:allergicReaction>
      <sp:CodedValue>
          <dcterms:title>Anaphylaxis</dcterms:title>
        <sp:code>
        <spcode:SNOMED rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/39579001">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Anaphylaxis</dcterms:title>
          <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
          <dcterms:identifier>39579001</dcterms:identifier>
        </spcode:SNOMED>
        </sp:code>
      </sp:CodedValue>

      </sp:allergicReaction>
      <sp:category>
      <sp:CodedValue>
          <dcterms:title>Drug allergy</dcterms:title>   
        <sp:code>
        <spcode:AllergyCategory rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/416098002">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Drug allergy</dcterms:title>   
          <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
          <dcterms:identifier>416098002</dcterms:identifier>
        </spcode:AllergyCategory>
        </sp:code>
      </sp:CodedValue>

      </sp:category>
   </sp:Allergy>
</rdf:RDF>
{% endhighlight  %}


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Allergy" class="external free" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">http://smartplatforms.org/terms#Allergy</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>allergicReaction</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Allergy" class="external text" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">sp:Allergy</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#SNOMED_code_RDF" title=""> SNOMED</a>
<p>Reaction associated with an allergy.  Code drawn from SNOMED-CT.
</p>
</td></tr>
<tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Allergy" class="external text" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">sp:Allergy</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%"><b>category</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Allergy" class="external text" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">sp:Allergy</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#AllergyCategory_code_RDF" title=""> AllergyCategory</a>
<p>Category of an allergy (food, drug, other substance).
</p>
</td></tr>
<tr>
<td width="30%">drugAllergen<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Allergy" class="external text" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">sp:Allergy</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#RxNorm_Ingredient_code_RDF" title=""> RxNorm_Ingredient</a>
<p>For drug allergies, an RxNorm Concept at the ingredient level (TTY='in').
</p>
</td></tr>
<tr>
<td width="30%">drugClassAllergen<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Allergy" class="external text" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">sp:Allergy</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#NDFRT_code_RDF" title=""> NDFRT</a>
<p>Class of allergen, e.g. Sulfonamides or ACE inhibitors.  RDF Code node with code drawn from
</p>
<pre>           NDF-RT: <a href="http://purl.bioontology.org/ontology/NDFRT/%7BNDFRT_ID%7D" class="external free" title="http://purl.bioontology.org/ontology/NDFRT/{NDFRT_ID}" rel="nofollow">http://purl.bioontology.org/ontology/NDFRT/{NDFRT_ID}</a>
</pre>
</td></tr>
<tr>
<td width="30%">foodAllergen<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Allergy" class="external text" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">sp:Allergy</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#UNII_code_RDF" title=""> UNII</a>
<p>Substance acting as a food or environmental allergen.  For environmental and food substance is a UNII:
</p>
</td></tr>
<tr>
<td width="30%"><b>severity</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Allergy" class="external text" title="http://smartplatforms.org/terms#Allergy" rel="nofollow">sp:Allergy</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#AllergySeverity_code_RDF" title=""> AllergySeverity</a>
<p>Severity of an allergy	
</p>
</td></tr>
</tbody></table>

##Allergy Exclusion RDF

Allergy Exclusion is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)


In clinical documentation, asserting that a patient has "No known allergies" is very different from not mentioning whether a patient has any allergies. It's a positive affirmation that a question was asked and answered. SMART models this information as an explicit clinical statement of the type AllergyExclusion.

While it might seem inelegant to expose explicit AllergyExclusion statements, this model is designed to combat a clinical modeling pattern where a single flag ("isNegated" or "negationIndicator" for example) negates the meaning of an entire statement. Interpreting statements in a world where negation flags exist can be tricky. Every app has to understand the subtlety of this flag -- and it's not always clear what it means to negate a statement with multiple parts. For more information about exclusion statements in clinical modeling, see [http://omowizard.wordpress.com/2011/06/06/unambiguous-data-positive-presence-positive-absence/](http://omowizard.wordpress.com/2011/06/06/unambiguous-data-positive-presence-positive-absence/). 


{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"> 

<sp:AllergyExclusion>
  <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
  <sp:allergyExclusionName>
      <sp:CodedValue>
        <dcterms:title>No known allergies</dcterms:title>
        <sp:code>
          <spcode:AllergyExclusion rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/160244002">
           <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
           <dcterms:title>No known allergies</dcterms:title>
           <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
           <dcterms:identifier>160244002</dcterms:identifier>
          </spcode:AllergyExclusion>
        </sp:code>
      </sp:CodedValue>
  </sp:allergyExclusionName>
</sp:AllergyExclusion>
</rdf:RDF>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#AllergyExclusion" class="external free" title="http://smartplatforms.org/terms#AllergyExclusion" rel="nofollow">http://smartplatforms.org/terms#AllergyExclusion</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>allergyExclusionName</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#AllergyExclusion" class="external text" title="http://smartplatforms.org/terms#AllergyExclusion" rel="nofollow">sp:AllergyExclusion</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#AllergyExclusion_code_RDF" title=""> AllergyExclusion</a>
<p>Nature of the allergy exclusion.
</p>
</td></tr>
<tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#AllergyExclusion" class="external text" title="http://smartplatforms.org/terms#AllergyExclusion" rel="nofollow">sp:AllergyExclusion</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
</tbody></table>

##Demographics RDF

Demographics is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF), [Person](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Person_RDF), [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF), [VCard](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)

In RDF/XML, patient Bob Odenkirk looks like this

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF
  xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:v="http://www.w3.org/2006/vcard/ns#"
  xmlns:foaf="http://xmlns.com/foaf/0.1/">
   <sp:Demographics>
     <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
     <v:n>
        <v:Name>
            <v:given-name>Bob</v:given-name>
            <v:additional-name>J</v:additional-name>
            <v:family-name>Odenkirk</v:family-name>
        </v:Name>
     </v:n>

     <v:adr>
        <v:Address>
          <rdf:type rdf:resource="http://www.w3.org/2006/vcard/ns#Home" />
          <rdf:type rdf:resource="http://www.w3.org/2006/vcard/ns#Pref" />

          <v:street-address>15 Main St</v:street-address>
          <v:extended-address>Apt 2</v:extended-address>
          <v:locality>Wonderland</v:locality>
          <v:region>OZ</v:region>
          <v:postal-code>54321</v:postal-code>
          <v:country>USA</v:country>
        </v:Address>
     </v:adr>

     <v:tel>
        <v:Tel>
          <rdf:type rdf:resource="http://www.w3.org/2006/vcard/ns#Home" />
          <rdf:type rdf:resource="http://www.w3.org/2006/vcard/ns#Pref" />
          <rdf:value>800-555-1212</rdf:value>
        </v:Tel>
     </v:tel>

     <v:tel>
        <v:Tel>
          <rdf:type rdf:resource="http://www.w3.org/2006/vcard/ns#Cell" />
          <rdf:value>800-555-1515</rdf:value>
        </v:Tel>
     </v:tel>

     <foaf:gender>male</foaf:gender>
     <v:bday>1959-12-25</v:bday>
     <v:email>bob.odenkirk@example.com</v:email>

     <sp:medicalRecordNumber>
       <sp:Code>
        <dcterms:title>My Hospital Record 2304575</dcterms:title> 
        <dcterms:identifier>2304575</dcterms:identifier> 
        <sp:system>My Hospital Record</sp:system> 
       </sp:Code>
     </sp:medicalRecordNumber>

   </sp:Demographics>
</rdf:RDF>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Demographics" class="external free" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">http://smartplatforms.org/terms#Demographics</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%">ethnicity<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>medicalRecordNumber</b><br><small>Required: 1 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"><a href="#Code_RDF" title=""> Code</a>
</td></tr>
<tr>
<td width="30%">preferredLanguage<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">race<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">adr<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"><a href="#Address_RDF" title=""> Address</a>
</td></tr>
<tr>
<td width="30%"><b>bday</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">email<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>n</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"><a href="#Name_RDF" title=""> Name</a>
</td></tr>
<tr>
<td width="30%">tel<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%"><a href="#Tel_RDF" title=""> Tel</a>
</td></tr>
<tr>
<td width="30%"><b>gender</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Demographics" class="external text" title="http://smartplatforms.org/terms#Demographics" rel="nofollow">sp:Demographics</a>]</small>
</td><td width="50%">A person's (administrative) gender.  This should consist of the string "male" or "female". [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##Encounter RDF

Encounter is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:v="http://www.w3.org/2006/vcard/ns#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"> 
      <sp:Encounter>
      <sp:belongsTo  rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
      <sp:startDate>2010-05-12T04:00:00Z</sp:startDate>
      <sp:endDate>2010-05-12T04:20:00Z</sp:endDate>
      <sp:encounterType>
       <sp:CodedValue>
         <dcterms:title>Ambulatory encounter</dcterms:title>
         <sp:code>
          <spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#ambulatory">
            <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
            <dcterms:title>Ambulatory encounter</dcterms:title>
            <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
            <dcterms:identifier>ambulatory</dcterms:identifier> 
          </spcode:EncounterType>       
         </sp:code>
       </sp:CodedValue>

      </sp:encounterType>
    </sp:Encounter>
</rdf:RDF>
{% endhighlight  %}


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Encounter" class="external free" title="http://smartplatforms.org/terms#Encounter" rel="nofollow">http://smartplatforms.org/terms#Encounter</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Encounter" class="external text" title="http://smartplatforms.org/terms#Encounter" rel="nofollow">sp:Encounter</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%"><b>encounterType</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Encounter" class="external text" title="http://smartplatforms.org/terms#Encounter" rel="nofollow">sp:Encounter</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#EncounterType_code_RDF" title=""> EncounterType</a>
<p>Type of encounter
</p>
</td></tr>
<tr>
<td width="30%">endDate<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Encounter" class="external text" title="http://smartplatforms.org/terms#Encounter" rel="nofollow">sp:Encounter</a>]</small>
</td><td width="50%">Date when encounter ended [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
<tr>
<td width="30%">facility<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Encounter" class="external text" title="http://smartplatforms.org/terms#Encounter" rel="nofollow">sp:Encounter</a>]</small>
</td><td width="50%"><a href="#Organization_RDF" title=""> Organization</a>
<p>Facility where encounter occurred
</p>
</td></tr>
<tr>
<td width="30%">provider<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Encounter" class="external text" title="http://smartplatforms.org/terms#Encounter" rel="nofollow">sp:Encounter</a>]</small>
</td><td width="50%"><a href="#Provider_RDF" title=""> Provider</a>
<p>Provider responsible for encounter
</p>
</td></tr>
<tr>
<td width="30%"><b>startDate</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Encounter" class="external text" title="http://smartplatforms.org/terms#Encounter" rel="nofollow">sp:Encounter</a>]</small>
</td><td width="50%">Date when encounter began [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
</tbody></table>

##Fulfillment RDF

Fulfillment is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)


{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:foaf="http://xmlns.com/foaf/0.1/"
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:v="http://www.w3.org/2006/vcard/ns#">
 <sp:Fulfillment>
    <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
    <dcterms:date>2010-05-12T04:00:00Z</dcterms:date>
    <sp:medication rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591/medications/123" />
    <sp:provider>
      <sp:Provider>
        <v:n>
          <v:Name>
           <v:given-name>Joshua</v:given-name>
           <v:family-name>Mandel</v:family-name>
          </v:Name>
        </v:n>
        <sp:npiNumber>5235235</sp:npiNumber>
        <sp:deaNumber>325555555</sp:deaNumber>
      </sp:Provider>
    </sp:provider>
    <sp:pharmacy>
      <sp:Pharmacy>
        <sp:ncpdpId>5235235</sp:ncpdpId>
            <v:organization-name>CVS #588</v:organization-name>
            <v:adr>
              <v:Address>
                <v:street-address>111 Lake Drive</v:street-address>
                <v:locality>WonderCity</v:locality>
                <v:postal-code>5555</v:postal-code>
                <v:country-name>Australia</v:country-name>
              </v:Address>
              </v:adr>
      </sp:Pharmacy>
    </sp:pharmacy>
    <sp:pbm>T00000000001011</sp:pbm>
    <sp:quantityDispensed>
      <sp:ValueAndUnit>
        <sp:value>60</sp:value>
        <sp:unit>{tablet}</sp:unit>
      </sp:ValueAndUnit>
    </sp:quantityDispensed>
    <sp:dispenseDaysSupply>30</sp:dispenseDaysSupply>
 </sp:Fulfillment>
</rdf:RDF>
{% endhighlight  %}


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Fulfillment" class="external free" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">http://smartplatforms.org/terms#Fulfillment</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>date</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%">Date on which medication was dispensed [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
<tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%"><b>dispenseDaysSupply</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%">The number of days' supply dispensed [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>medication</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%"><a href="#Medication_RDF" title=""> Medication</a>
</td></tr>
<tr>
<td width="30%">pbm<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%">The PBM providing payment for medications [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">pharmacy<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%"><a href="#Pharmacy_RDF" title=""> Pharmacy</a>
<p>The pharmacy that dispensed the medication
</p>
</td></tr>
<tr>
<td width="30%">provider<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%"><a href="#Provider_RDF" title=""> Provider</a>
<p>Clinician who prescribed the medication
</p>
</td></tr>
<tr>
<td width="30%">quantityDispensed<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Fulfillment" class="external text" title="http://smartplatforms.org/terms#Fulfillment" rel="nofollow">sp:Fulfillment</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p>Quantity dispensed, with units
</p>
</td></tr>
</tbody></table>

##Immunization RDF

Immunization is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)


Explicit record of an immunization given or not given to the patient. 

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"> 

   <sp:Immunization>
	  <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
	  <dcterms:date>2010-05-12T04:00:00Z</dcterms:date>

	  <sp:administrationStatus>
	    <sp:CodedValue>
	      <dcterms:title>Not Administered</dcterms:title>
	      <sp:code>
	        <sp:Code rdf:about="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#notAdministered">
	          <rdf:type rdf:resource="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" /> 
	          <dcterms:title>Not Administered</dcterms:title>
	          <sp:system>http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#</sp:system>
	          <dcterms:identifier>notAdministered</dcterms:identifier>
	        </sp:Code>
	      </sp:code>
	    </sp:CodedValue>
	  </sp:administrationStatus>

	  <sp:refusalReason>
	    <sp:CodedValue>
	      <dcterms:title>Allergy to vaccine/vaccine components, or allergy to eggs</dcterms:title>
	      <sp:code>
	        <sp:Code rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#allergy">
	          <rdf:type rdf:resource="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" /> 
	          <dcterms:title>Allergy to vaccine/vaccine components, or allergy to eggs</dcterms:title>
	          <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
	          <dcterms:identifier>allergy</dcterms:identifier>
	        </sp:Code>
	      </sp:code>
	    </sp:CodedValue>
	  </sp:refusalReason>

	  <sp:productName>
	    <sp:CodedValue>
	      <dcterms:title>typhoid, oral</dcterms:title>
	      <sp:code>
	        <sp:Code rdf:about="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#25">
	          <rdf:type rdf:resource="http://smartplatforms.org/terms/codes/ImmunizationProduct" /> 
	          <dcterms:title>typhoid, oral</dcterms:title>
	          <sp:system>http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#</sp:system>
	          <dcterms:identifier>25</dcterms:identifier>
	        </sp:Code>
	      </sp:code>
	    </sp:CodedValue>
	  </sp:productName>

	  <sp:productClass>
	    <sp:CodedValue>
	      <dcterms:title>TYPHOID</dcterms:title>
	      <sp:code>
	        <sp:Code rdf:about="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#TYPHOID">
	          <rdf:type rdf:resource="http://smartplatforms.org/terms/codes/ImmunizationClass" /> 
	          <dcterms:title>TYPHOID</dcterms:title>
	          <sp:system>http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#</sp:system>
	          <dcterms:identifier>TYPHOID</dcterms:identifier>
	        </sp:Code>
	      </sp:code>
	    </sp:CodedValue>
	  </sp:productClass>

   </sp:Immunization>
</rdf:RDF>

{% endhighlight  %}



<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Immunization" class="external free" title="http://smartplatforms.org/terms#Immunization" rel="nofollow">http://smartplatforms.org/terms#Immunization</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>date</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Immunization" class="external text" title="http://smartplatforms.org/terms#Immunization" rel="nofollow">sp:Immunization</a>]</small>
</td><td width="50%">ISO8601 formatted date and time when the medication was administered or offered. [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
<tr>
<td width="30%"><b>administrationStatus</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Immunization" class="external text" title="http://smartplatforms.org/terms#Immunization" rel="nofollow">sp:Immunization</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#ImmunizationAdministrationStatus_code_RDF" title=""> ImmunizationAdministrationStatus</a>
</td></tr>
<tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Immunization" class="external text" title="http://smartplatforms.org/terms#Immunization" rel="nofollow">sp:Immunization</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%">productClass<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Immunization" class="external text" title="http://smartplatforms.org/terms#Immunization" rel="nofollow">sp:Immunization</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#ImmunizationClass_code_RDF" title=""> ImmunizationClass</a>
<p>coded name for the product class, according to the set of codes in 
the CDC Vaccine Groups controlled vocabulary.  For example, a class code
 meaning 'Rotavirus' would be assigned for a specific product such as 
Rotarix product.
</p><p><br>
productClass codes are drawn from the CDC's Vaccine Group vocabulary.  URIs are of the form:
<a href="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#code" class="external free" title="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#code" rel="nofollow">http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#code</a>
</p><p><br>
For example, the URI for the ROTAVIRUS code is:
<a href="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#ROTAVIRUS" class="external free" title="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#ROTAVIRUS" rel="nofollow">http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#ROTAVIRUS</a>
</p>
</td></tr>
<tr>
<td width="30%"><b>productName</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Immunization" class="external text" title="http://smartplatforms.org/terms#Immunization" rel="nofollow">sp:Immunization</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#ImmunizationProduct_code_RDF" title=""> ImmunizationProduct</a>
<p>coded describing the product according to the set of codes in the CVX
 for immunizationa controlled vocabulary.  CVX Code URIs should be 
represented as:
</p><p><a href="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#%7Bcode%7D" class="external free" title="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#{code}" rel="nofollow">http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#{code}</a>
</p><p><br>
For exampe, the code for "adenovirus, type 4" is 54, and its URI is:
</p><p><a href="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#54" class="external free" title="http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#54" rel="nofollow">http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#54</a>
</p>
</td></tr>
<tr>
<td width="30%">refusalReason<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Immunization" class="external text" title="http://smartplatforms.org/terms#Immunization" rel="nofollow">sp:Immunization</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#ImmunizationRefusalReason_code_RDF" title=""> ImmunizationRefusalReason</a>
<p>If the administration status indicates this vaccination was refused, 
refusalReason is a CodedValue whose code is belongs to a controlled 
vocabulary of refusal reasons.
</p>
</td></tr>
</tbody></table>

##Lab Result RDF

Lab Result is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)


In RDF/XML, a serum sodium result looks like this 


{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF xmlns:sp="http://smartplatforms.org/terms#" 
  xmlns:dcterms="http://purl.org/dc/terms/" 
  xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#" 
  xmlns:foaf="http://xmlns.com/foaf/0.1/"
  xmlns:v="http://www.w3.org/2006/vcard/ns#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/">

  <sp:LabResult>
      <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
      <sp:labName>
    <sp:CodedValue>
            <dcterms:title>Serum sodium</dcterms:title>
            <sp:provenance>
              <sp:CodeProvenance>
                <sp:sourceCode rdf:resource="http://my.local.coding.system/01234" />
                <dcterms:title>Random blood sodium level</dcterms:title>
                <sp:translationFidelity rdf:resource="http://smartplatforms.org/terms/codes/TranslationFidelity#verified"/>
              </sp:CodeProvenance>
            </sp:provenance>
        <sp:code>
              <spcode:LOINC rdf:about="http://purl.bioontology.org/ontology/LNC/2951-2">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Serum sodium</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>2951-2</dcterms:identifier> 
              </spcode:LOINC>    
          </sp:code>
        </sp:CodedValue>
      </sp:labName>

      <sp:quantitativeResult>
        <sp:QuantitativeResult>
          <sp:valueAndUnit>
            <sp:ValueAndUnit>
              <sp:value>140</sp:value>
              <sp:unit>mEq/L</sp:unit>
            </sp:ValueAndUnit>
          </sp:valueAndUnit>
          <sp:normalRange>
             <sp:ValueRange>
              <sp:minimum>
                <sp:ValueAndUnit>
                   <sp:value>135</sp:value>
                   <sp:unit>mEq/L</sp:unit>
                </sp:ValueAndUnit>
              </sp:minimum>
              <sp:maximum>
                <sp:ValueAndUnit>
                   <sp:value>145</sp:value>
                   <sp:unit>mEq/L</sp:unit>
                </sp:ValueAndUnit>
              </sp:maximum>
             </sp:ValueRange>
          </sp:normalRange>
          <sp:nonCriticalRange>
             <sp:ValueRange>
              <sp:minimum>
                <sp:ValueAndUnit>
                   <sp:value>120</sp:value>
                   <sp:unit>mEq/L</sp:unit>
                </sp:ValueAndUnit>
              </sp:minimum>
              <sp:maximum>
                <sp:ValueAndUnit>
                   <sp:value>155</sp:value>
                   <sp:unit>mEq/L</sp:unit>
                </sp:ValueAndUnit>
              </sp:maximum>
             </sp:ValueRange>
          </sp:nonCriticalRange>
        </sp:QuantitativeResult>
      </sp:quantitativeResult>
      <sp:accessionNumber>AC09205823577</sp:accessionNumber>
      <sp:labStatus>
    <sp:CodedValue>
      <dcterms:title>Final results: complete and verified</dcterms:title>      
      <sp:code>
        <spcode:LabResultStatus rdf:about="http://smartplatforms.org/terms/codes/LabStatus#final">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Final</dcterms:title>      
          <sp:system>http://smartplatforms.org/terms/codes/LabStatus#</sp:system>
          <dcterms:identifier>final</dcterms:identifier> 
        </spcode:LabResultStatus>    
      </sp:code>
    </sp:CodedValue>

      </sp:labStatus>
      <sp:abnormalInterpretation>
    <sp:CodedValue>
      <dcterms:title>Normal</dcterms:title>      
      <sp:code>
        <spcode:LabResultInterpretation rdf:about="http://smartplatforms.org/terms/codes/LabResultInterpretation#normal">
          <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
          <dcterms:title>Normal</dcterms:title>      
          <sp:system>http://smartplatforms.org/terms/codes/LabResultInterpretation#</sp:system>
          <dcterms:identifier>normal</dcterms:identifier> 
        </spcode:LabResultInterpretation>    
      </sp:code>
    </sp:CodedValue>

      </sp:abnormalInterpretation>
      <sp:specimenCollected>
        <sp:Attribution>
          <sp:startDate>2010-12-27T17:00:00</sp:startDate>
        </sp:Attribution>
      </sp:specimenCollected>
      <sp:notes>Blood sample appears to have hemolyzed</sp:notes>
   </sp:LabResult>
</rdf:RDF>
{% endhighlight  %}




<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#LabResult" class="external free" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">http://smartplatforms.org/terms#LabResult</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">abnormalInterpretation<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#LabResultInterpretation_code_RDF" title=""> LabResultInterpretation</a>
<p>Abnormal interpretation status for this lab
</p>
</td></tr>
<tr>
<td width="30%">accessionNumber<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%">External accession number for a lab result [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%"><b>labName</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#LOINC_code_RDF" title=""> LOINC</a>
<p>LOINC Coded Value for result (e.g. with title='Serum Sodium' and code=<a href="http://purl.bioontology.org/ontology/LNC/2951-2" class="external free" title="http://purl.bioontology.org/ontology/LNC/2951-2" rel="nofollow">http://purl.bioontology.org/ontology/LNC/2951-2</a>
</p>
</td></tr>
<tr>
<td width="30%">labStatus<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#LabResultStatus_code_RDF" title=""> LabResultStatus</a>
<p>Workflow status of this lab value (e.g. "finalized")
</p>
</td></tr>
<tr>
<td width="30%">narrativeResult<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%"><a href="#NarrativeResult_RDF" title=""> NarrativeResult</a>
<p>Narrative result, if any.
</p>
</td></tr>
<tr>
<td width="30%">notes<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%">Free-text notes about this result. [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">quantitativeResult<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%"><a href="#QuantitativeResult_RDF" title=""> QuantitativeResult</a>
<p>Qualitative result, if any
</p>
</td></tr>
<tr>
<td width="30%">specimenCollected<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#LabResult" class="external text" title="http://smartplatforms.org/terms#LabResult" rel="nofollow">sp:LabResult</a>]</small>
</td><td width="50%"><a href="#Attribution_RDF" title=""> Attribution</a>
<p>Attribution specificying when specimen was collected and who was responsible for them.
</p>
</td></tr>
</tbody></table>

##Medication RDF

Medication is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)


The SMART medication type expresses a medication at the level of an RxNorm branded or generic drug concept (e.g. "20 mg generic loratadine" or "20 mg brand-name claritin"). A medication may include start- and end-dates, as well as a free-text "instructions" field describing how it should be taken. When the instructions are simple enough, we also represent them in a structured way, aiming to capture about 80% of outpatient medication dosing schedules. A very simple semantic structure defines how much to take ("quantity") and how often ("frequency"). Both quantity and frequency are defined with expressions from [The Unified Code for Units of Measure](http://www.unitsofmeasure.org/), or UCUM (see below).

In RDF/XML notation, a patient on oral amitriptyline 50 mg tablets might provide the following RDF sub-graph as part of a medication list

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"
  xmlns:dcterms="http://purl.org/dc/terms/">
   <sp:Medication>
      <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
      <sp:drugName>
      <sp:CodedValue>
          <dcterms:title>AMITRIPTYLINE HCL 50 MG TAB</dcterms:title>
          <sp:code>
            <spcode:RxNorm_Semantic rdf:about="http://purl.bioontology.org/ontology/RXNORM/856845">
              <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
              <sp:system>http://purl.bioontology.org/ontology/RXNORM/</sp:system>
              <dcterms:identifier>856845</dcterms:identifier>
              <dcterms:title>AMITRIPTYLINE HCL 50 MG TAB</dcterms:title>
            </spcode:RxNorm_Semantic>    
          </sp:code>
      </sp:CodedValue>
      </sp:drugName>
      <sp:startDate>2007-03-14</sp:startDate>
      <sp:endDate>2007-08-14</sp:endDate>
      <sp:instructions>Take two tablets twice daily as needed for pain</sp:instructions>
      <sp:quantity>
          <sp:ValueAndUnit>
            <sp:value>2</sp:value>
            <sp:unit>{tablet}</sp:unit>
          </sp:ValueAndUnit>
      </sp:quantity>
      <sp:frequency>
          <sp:ValueAndUnit>
            <sp:value>2</sp:value>
            <sp:unit>/d</sp:unit>
          </sp:ValueAndUnit>
      </sp:frequency>
   </sp:Medication>
</rdf:RDF>

{% endhighlight  %}



<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Medication" class="external free" title="http://smartplatforms.org/terms#Medication" rel="nofollow">http://smartplatforms.org/terms#Medication</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%"><b>drugName</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#RxNorm_Semantic_code_RDF" title=""> RxNorm_Semantic</a>
<p>RxNorm Concept ID for this medication.  Note: the RxNorm CUI for a 
SMART medication should have one of the following four types:  SCD 
(Semantic Clinical Drug), SBD (Semantic Branded Drug), GPCK (Generic 
Pack), BPCK (Brand Name Pack).   Restricting medications to these four 
RxNorm types can also be expressed as "TTY in 
('SCD','SBD','GPCK','BPCK')" -- and this restriction ensures that SMART 
medications use concepts of the appropriate specificity:  concepts like 
"650 mg generic acetaminophen" or "20 mg brand-name Claritin".   Please 
note that SMART medications do not include explicit structured data 
about pill strength, concentration, or precise ingredients.  These data 
are available through RxNorm, including through the free <a href="http://rxnav.nlm.nih.gov/RxNormRestAPI.html" class="external text" title="http://rxnav.nlm.nih.gov/RxNormRestAPI.html" rel="nofollow">RxNav REST API</a>.  Code element with code drawn from <a href="http://purl.bioontology.org/ontology/RXNORM/%7Brxcui%7D" class="external free" title="http://purl.bioontology.org/ontology/RXNORM/{rxcui}" rel="nofollow">http://purl.bioontology.org/ontology/RXNORM/{rxcui}</a>
</p>
</td></tr>
<tr>
<td width="30%">endDate<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%">
<p>	When the patient stopped taking a medication 
	 [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</p>
</td></tr>
<tr>
<td width="30%">frequency<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p><br>
For a medication with a simple dosing schedule, record how often to take the medication.  The frequency should be recorded as a <a href="http://www.unitsofmeasure.org/" class="external text" title="http://www.unitsofmeasure.org/" rel="nofollow">UCUM</a>
 expression.  In this case we use a restricted subset of UCUM that 
defines the following units only:  "/d" (per day), "/wk" (per week), 
"/mo" (per month).  For example, you would express the concept of "TID" 
or "three times daily" as a ValueAndUnit node with quantity="3", 
unit="/d". 
</p>
</td></tr>
<tr>
<td width="30%">fulfillment<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%"><a href="#Fulfillment_RDF" title=""> Fulfillment</a>
<p><br>
A single fulfillment event (that is, the medication was dispensed to the patient by a pharmacy)
</p>
</td></tr>
<tr>
<td width="30%"><b>instructions</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%">
<p>	Clinician-supplied instructions from the prescription signature 
	 [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</p>
</td></tr>
<tr>
<td width="30%">provenance<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%"><a href="#Code_RDF" title=""> Code</a> where code comes from <a href="#MedicationProvenance_code_RDF" title=""> MedicationProvenance</a>
</td></tr>
<tr>
<td width="30%">quantity<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p><br>
For a medication with a simple dosing schedule, record the amount to 
take with each administration.  The quantity should be recorded as a <a href="http://www.unitsofmeasure.org/" class="external text" title="http://www.unitsofmeasure.org/" rel="nofollow">UCUM</a>
 expression.  For some medications, the appoporiate quantity will be a 
volume (e.g. "5 mL" of an oral acetaminophen solution).  For other 
medications, the appropriate quantity may be expresses in terms of 
tablets, puffs, or actuations:  UCUM call these "non-units", and they 
should be written inside of curly braces to avoid confusion.  For 
example, you would express "1 tablet" as a ValueAndUnit node with 
quantity="1", unit="{tablet}".
</p>
</td></tr>
<tr>
<td width="30%"><b>startDate</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Medication" class="external text" title="http://smartplatforms.org/terms#Medication" rel="nofollow">sp:Medication</a>]</small>
</td><td width="50%">
<p>	When the patient started taking a medication 
	 [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</p>
</td></tr>
</tbody></table>

##Problem RDF

Problem is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"> 
    <sp:Problem>
      <sp:belongsTo rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
      <sp:problemName>
      <sp:CodedValue>
          <dcterms:title>Backache (finding)</dcterms:title>      
          <sp:code>
            <spcode:SNOMED rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/161891005">
              <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
              <dcterms:title>Backache (finding)</dcterms:title>      
              <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
              <dcterms:identifier>161891005</dcterms:identifier> 
            </spcode:SNOMED>
          </sp:code>
      </sp:CodedValue>
      </sp:problemName>
      <sp:startDate>2007-06-12</sp:startDate>
      <sp:endDate>2007-08-01</sp:endDate>
    </sp:Problem>
</rdf:RDF>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Problem" class="external free" title="http://smartplatforms.org/terms#Problem" rel="nofollow">http://smartplatforms.org/terms#Problem</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Problem" class="external text" title="http://smartplatforms.org/terms#Problem" rel="nofollow">sp:Problem</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%">endDate<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Problem" class="external text" title="http://smartplatforms.org/terms#Problem" rel="nofollow">sp:Problem</a>]</small>
</td><td width="50%">Date on which problem resolved, if any. [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
<tr>
<td width="30%">notes<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Problem" class="external text" title="http://smartplatforms.org/terms#Problem" rel="nofollow">sp:Problem</a>]</small>
</td><td width="50%">Additional notes about the problem [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>problemName</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Problem" class="external text" title="http://smartplatforms.org/terms#Problem" rel="nofollow">sp:Problem</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#SNOMED_code_RDF" title=""> SNOMED</a>
<p>SNOMED-CT Concept for the problem
</p>
</td></tr>
<tr>
<td width="30%"><b>startDate</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Problem" class="external text" title="http://smartplatforms.org/terms#Problem" rel="nofollow">sp:Problem</a>]</small>
</td><td width="50%">Date on which problem began [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
</tbody></table>

##SMART Statement RDF

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Statement" class="external free" title="http://smartplatforms.org/terms#Statement" rel="nofollow">http://smartplatforms.org/terms#Statement</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Statement" class="external text" title="http://smartplatforms.org/terms#Statement" rel="nofollow">sp:Statement</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
</tbody></table>

##User Preferences RDF

##VitalSigns RDF

VitalSigns is a subtype of and inherits properties from [SMART Statement](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#SMART_Statement_RDF)

{% highlight html %}
<?xml version="1.0" encoding="utf-8"?>
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
  xmlns:sp="http://smartplatforms.org/terms#"
  xmlns:foaf="http://xmlns.com/foaf/0.1/"
  xmlns:dc="http://purl.org/dc/elements/1.1/"
  xmlns:dcterms="http://purl.org/dc/terms/"
  xmlns:v="http://www.w3.org/2006/vcard/ns#"
  xmlns:spcode="http://smartplatforms.org/terms/codes/"> 

  <sp:VitalSigns>
    <sp:belongsTo  rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
    <dcterms:date>2010-05-12T04:00:00Z</dcterms:date>
    <sp:encounter>
      <sp:Encounter>
        <sp:belongsTo  rdf:resource="http://sandbox-api.smartplatforms.org/records/2169591" />
        <sp:startDate>2010-05-12T04:00:00Z</sp:startDate>
        <sp:endDate>2010-05-12T04:20:00Z</sp:endDate>
        <sp:encounterType>
          <sp:CodedValue>
            <dcterms:title>Ambulatory encounter</dcterms:title>
            <sp:code>
              <spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#ambulatory">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Ambulatory encounter</dcterms:title>
                <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
                <dcterms:identifier>ambulatory</dcterms:identifier> 
              </spcode:EncounterType>       
            </sp:code>
          </sp:CodedValue>
        </sp:encounterType>
      </sp:Encounter>    
    </sp:encounter>
    <sp:height>
      <sp:VitalSign>
        <sp:vitalName>
          <sp:CodedValue>
            <dcterms:title>Body height</dcterms:title>
            <sp:code>
              <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8302-2">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Body height</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>8302-2</dcterms:identifier> 
              </spcode:VitalSign>    
            </sp:code>
          </sp:CodedValue>
        </sp:vitalName>
        <sp:value>1.80</sp:value>
        <sp:unit>m</sp:unit>
      </sp:VitalSign>
    </sp:height>
    <sp:weight>
      <sp:VitalSign>
        <sp:vitalName>
          <sp:CodedValue>
            <dcterms:title>Body weight</dcterms:title>
            <sp:code>    
              <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/3141-9">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Body weight</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>3141-9</dcterms:identifier> 
              </spcode:VitalSign>
            </sp:code>
          </sp:CodedValue>
        </sp:vitalName>
        <sp:value>70.8</sp:value>
        <sp:unit>kg</sp:unit>
      </sp:VitalSign>
    </sp:weight>
    <sp:bodyMassIndex>
      <sp:VitalSign>
        <sp:vitalName>
          <sp:CodedValue>
            <dcterms:title>Body mass index</dcterms:title>
            <sp:code>
              <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/39156-5">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Body mass index</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>39156-5</dcterms:identifier> 
              </spcode:VitalSign>        
            </sp:code>
          </sp:CodedValue>
        </sp:vitalName>
        <sp:value>21.8</sp:value>
        <sp:unit>kg/m2</sp:unit>
      </sp:VitalSign>
    </sp:bodyMassIndex>
    <sp:respiratoryRate>
      <sp:VitalSign>
        <sp:vitalName>
          <sp:CodedValue>
            <dcterms:title>Respiration rate</dcterms:title>
            <sp:code>
              <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/9279-1">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Respiration rate</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>9279-1</dcterms:identifier> 
              </spcode:VitalSign>    
            </sp:code>
          </sp:CodedValue>
        </sp:vitalName>
        <sp:value>16</sp:value>
        <sp:unit>{breaths}/min</sp:unit>
      </sp:VitalSign>
    </sp:respiratoryRate>
    <sp:heartRate>
      <sp:VitalSign>
        <sp:vitalName>
          <sp:CodedValue>
            <dcterms:title>Heart rate</dcterms:title>
            <sp:code>
              <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8867-4">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Heart rate</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>8867-4</dcterms:identifier> 
              </spcode:VitalSign>
            </sp:code>
          </sp:CodedValue>
        </sp:vitalName>
        <sp:value>70</sp:value>
        <sp:unit>{beats}/min</sp:unit>
      </sp:VitalSign>
    </sp:heartRate>
    <sp:oxygenSaturation>
      <sp:VitalSign>
        <sp:vitalName>
          <sp:CodedValue>
            <dcterms:title>Oxygen saturation</dcterms:title>
            <sp:code>
              <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/2710-2">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Oxygen saturation</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>2710-2</dcterms:identifier> 
              </spcode:VitalSign>        
            </sp:code>
          </sp:CodedValue>
        </sp:vitalName>
        <sp:value>99</sp:value>
        <sp:unit>%{HemoglobinSaturation}</sp:unit>
      </sp:VitalSign>
    </sp:oxygenSaturation>
    <sp:temperature>
      <sp:VitalSign>
        <sp:vitalName>
          <sp:CodedValue>
            <dcterms:title>Body temperature</dcterms:title>
            <sp:code>
              <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8310-5">
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Body temperature</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                <dcterms:identifier>8310-5</dcterms:identifier> 
              </spcode:VitalSign>        
            </sp:code>
          </sp:CodedValue>
        </sp:vitalName>
        <sp:value>37</sp:value>
        <sp:unit>Cel</sp:unit>
      </sp:VitalSign>
    </sp:temperature>
    <sp:bloodPressure>
      <sp:BloodPressure>
        <sp:systolic>
          <sp:VitalSign>
            <sp:vitalName>
              <sp:CodedValue>
                <dcterms:title>Intravascular systolic</dcterms:title>
                <sp:code>
                  <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8480-6">
                    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                    <dcterms:title>Intravascular systolic</dcterms:title>
                    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                    <dcterms:identifier>8480-6</dcterms:identifier> 
                  </spcode:VitalSign>        
                </sp:code>
              </sp:CodedValue>
            </sp:vitalName>
            <sp:value>132</sp:value>
            <sp:unit>mm[Hg]</sp:unit>
          </sp:VitalSign>
        </sp:systolic>
        <sp:diastolic>
          <sp:VitalSign>
            <sp:vitalName>
              <sp:CodedValue>
                <dcterms:title>Intravascular diastolic</dcterms:title>
                <sp:code>
                  <spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8462-4">
                    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                    <dcterms:title>Intravascular diastolic</dcterms:title>
                    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
                    <dcterms:identifier>8462-4</dcterms:identifier> 
                  </spcode:VitalSign>        
                </sp:code>
              </sp:CodedValue>
            </sp:vitalName>
            <sp:value>82</sp:value>
            <sp:unit>mm[Hg]</sp:unit>
          </sp:VitalSign>
        </sp:diastolic>
        <sp:bodyPosition>
          <sp:CodedValue>
            <dcterms:title>Sitting</dcterms:title>
            <sp:code>
              <spcode:BloodPressureBodyPosition rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/33586001" >
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Sitting</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
                <dcterms:identifier>33586001</dcterms:identifier> 
              </spcode:BloodPressureBodyPosition>        
            </sp:code>
          </sp:CodedValue>
        </sp:bodyPosition>
        <sp:bodySite>
          <sp:CodedValue>
            <dcterms:title>Right arm</dcterms:title>
            <sp:code>
              <spcode:BloodPressureBodySite rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/368209003" >
                <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
                <dcterms:title>Right arm</dcterms:title>
                <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
                <dcterms:identifier>368209003</dcterms:identifier> 
              </spcode:BloodPressureBodySite>        
            </sp:code>
          </sp:CodedValue>
        </sp:bodySite>
      </sp:BloodPressure>
    </sp:bloodPressure>
  </sp:VitalSigns>
</rdf:RDF>

{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#VitalSigns" class="external free" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">http://smartplatforms.org/terms#VitalSigns</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>date</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%">Date + time when vital signs were recorded [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
<tr>
<td width="30%"><b>belongsTo</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#Medical_Record_RDF" title=""> Medical Record</a>
<p>The medical record URI to which a clinical statement belongs.  Each 
clinical statement points back to its medical record so that it can be 
treated in isolation.
</p>
</td></tr>
<tr>
<td width="30%">bloodPressure<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#BloodPressure_RDF" title=""> BloodPressure</a>
<p>Patient's systolic + diastolic Blood Pressure in mmHg, with optinal position coding
</p>
</td></tr>
<tr>
<td width="30%">bodyMassIndex<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: kg/m2
<p>Patient's Body Mass Index.  
</p>
</td></tr>
<tr>
<td width="30%"><b>encounter</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#Encounter_RDF" title=""> Encounter</a>
<p>Encounter at which vital signs were measured. This should specify a date and encounter type, at minimum.
</p>
</td></tr>
<tr>
<td width="30%">heartRate<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: {beats}/min
<p>Patient's Heart Rate per minute. 
</p>
</td></tr>
<tr>
<td width="30%">height<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: m
<p>Patient's height in meters.  
</p>
</td></tr>
<tr>
<td width="30%">oxygenSaturation<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: %{HemoglobinSaturation}
<p>Patient's oxygen saturation in percent.  
</p>
</td></tr>
<tr>
<td width="30%">respiratoryRate<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: {breaths}/min
<p>Patient's Respiratory Rate per minute. 
</p>
</td></tr>
<tr>
<td width="30%">temperature<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: Cel
<p>Patient's Temperature in Celcius. 
</p>
</td></tr>
<tr>
<td width="30%">weight<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSigns" class="external text" title="http://smartplatforms.org/terms#VitalSigns" rel="nofollow">sp:VitalSigns</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: kg
<p>Patient's weight in kg. 
</p>
</td></tr>
</tbody></table>

#Component Types
##Address RDF

Address is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF)

* Preferred Status (v Pref)
* Home address (v Home)
* Work address (v Work)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://www.w3.org/2006/vcard/ns#Address" class="external free" title="http://www.w3.org/2006/vcard/ns#Address" rel="nofollow">http://www.w3.org/2006/vcard/ns#Address</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">country-name<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Address" class="external text" title="http://www.w3.org/2006/vcard/ns#Address" rel="nofollow">v:Address</a>]</small>
</td><td width="50%">Country name [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">extended-address<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Address" class="external text" title="http://www.w3.org/2006/vcard/ns#Address" rel="nofollow">v:Address</a>]</small>
</td><td width="50%">City Name [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">locality<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Address" class="external text" title="http://www.w3.org/2006/vcard/ns#Address" rel="nofollow">v:Address</a>]</small>
</td><td width="50%">City Name [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">postal-code<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Address" class="external text" title="http://www.w3.org/2006/vcard/ns#Address" rel="nofollow">v:Address</a>]</small>
</td><td width="50%">Postal code [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">region<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Address" class="external text" title="http://www.w3.org/2006/vcard/ns#Address" rel="nofollow">v:Address</a>]</small>
</td><td width="50%">e.g. state abbreviation [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">street-address<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Address" class="external text" title="http://www.w3.org/2006/vcard/ns#Address" rel="nofollow">v:Address</a>]</small>
</td><td width="50%">Street address [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##Attribution RDF

Attribution is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Attribution" class="external free" title="http://smartplatforms.org/terms#Attribution" rel="nofollow">http://smartplatforms.org/terms#Attribution</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">endDate<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Attribution" class="external text" title="http://smartplatforms.org/terms#Attribution" rel="nofollow">sp:Attribution</a>]</small>
</td><td width="50%">End Time of attributed event (if instantaneous, this is not provided) [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
<tr>
<td width="30%">participant<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Attribution" class="external text" title="http://smartplatforms.org/terms#Attribution" rel="nofollow">sp:Attribution</a>]</small>
</td><td width="50%"><a href="#Participant_RDF" title=""> Participant</a>
<p>Participant in Attribution
</p>
</td></tr>
<tr>
<td width="30%">startDate<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Attribution" class="external text" title="http://smartplatforms.org/terms#Attribution" rel="nofollow">sp:Attribution</a>]</small>
</td><td width="50%">Start Time of attributed event (if instantaneous, this is the only time) [<a href="http://www.w3.org/2001/XMLSchema#dateTime" class="external text" title="http://www.w3.org/2001/XMLSchema#dateTime" rel="nofollow">xsd:dateTime</a>]
</td></tr>
</tbody></table>

##BloodPressure RDF

BloodPressure is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#BloodPressure" class="external free" title="http://smartplatforms.org/terms#BloodPressure" rel="nofollow">http://smartplatforms.org/terms#BloodPressure</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">bodyPosition<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#BloodPressure" class="external text" title="http://smartplatforms.org/terms#BloodPressure" rel="nofollow">sp:BloodPressure</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#BloodPressureBodyPosition_code_RDF" title=""> BloodPressureBodyPosition</a>
<p>Position of patient when blood pressure was recorded
</p>
</td></tr>
<tr>
<td width="30%">bodySite<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#BloodPressure" class="external text" title="http://smartplatforms.org/terms#BloodPressure" rel="nofollow">sp:BloodPressure</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#BloodPressureBodySite_code_RDF" title=""> BloodPressureBodySite</a>
<p>Site on patient's body where blood pressure was recorded
</p>
</td></tr>
<tr>
<td width="30%"><b>diastolic</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#BloodPressure" class="external text" title="http://smartplatforms.org/terms#BloodPressure" rel="nofollow">sp:BloodPressure</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: mm[Hg]
<p>diastolic blood pressure in mmHG. 
</p>
</td></tr>
<tr>
<td width="30%">method<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#BloodPressure" class="external text" title="http://smartplatforms.org/terms#BloodPressure" rel="nofollow">sp:BloodPressure</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#BloodPressureMethod_code_RDF" title=""> BloodPressureMethod</a>
<p>Method by which blood pressure was recorded.
</p>
</td></tr>
<tr>
<td width="30%"><b>systolic</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#BloodPressure" class="external text" title="http://smartplatforms.org/terms#BloodPressure" rel="nofollow">sp:BloodPressure</a>]</small>
</td><td width="50%"><a href="#VitalSign_RDF" title=""> VitalSignwhere</a> unit has value: mm[Hg]
<p>systolic blood pressure in mmHG.
</p>
</td></tr>
</tbody></table>

##Code RDF

Code is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF), [DataType](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#DataType_RDF)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Code" class="external free" title="http://smartplatforms.org/terms#Code" rel="nofollow">http://smartplatforms.org/terms#Code</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Code" class="external text" title="http://smartplatforms.org/terms#Code" rel="nofollow">sp:Code</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Code" class="external text" title="http://smartplatforms.org/terms#Code" rel="nofollow">sp:Code</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Code" class="external text" title="http://smartplatforms.org/terms#Code" rel="nofollow">sp:Code</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##CodeProvenance RDF

CodeProvenance is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#CodeProvenance" class="external free" title="http://smartplatforms.org/terms#CodeProvenance" rel="nofollow">http://smartplatforms.org/terms#CodeProvenance</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#CodeProvenance" class="external text" title="http://smartplatforms.org/terms#CodeProvenance" rel="nofollow">sp:CodeProvenance</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>sourceCode</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#CodeProvenance" class="external text" title="http://smartplatforms.org/terms#CodeProvenance" rel="nofollow">sp:CodeProvenance</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2001/XMLSchema#anyURI" class="external text" title="http://www.w3.org/2001/XMLSchema#anyURI" rel="nofollow">xsd:anyURI</a>]
</td></tr>
<tr>
<td width="30%">translationFidelity<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#CodeProvenance" class="external text" title="http://smartplatforms.org/terms#CodeProvenance" rel="nofollow">sp:CodeProvenance</a>]</small>
</td><td width="50%"><a href="#TranslationFidelity_code_RDF" title=""> TranslationFidelity code</a>
</td></tr>
</tbody></table>

##Coded Value RDF

Coded Value is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF), [DataType](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#DataType_RDF)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#CodedValue" class="external free" title="http://smartplatforms.org/terms#CodedValue" rel="nofollow">http://smartplatforms.org/terms#CodedValue</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#CodedValue" class="external text" title="http://smartplatforms.org/terms#CodedValue" rel="nofollow">sp:CodedValue</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>code</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#CodedValue" class="external text" title="http://smartplatforms.org/terms#CodedValue" rel="nofollow">sp:CodedValue</a>]</small>
</td><td width="50%"><a href="#Code_RDF" title=""> Code</a>
</td></tr>
<tr>
<td width="30%">provenance<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#CodedValue" class="external text" title="http://smartplatforms.org/terms#CodedValue" rel="nofollow">sp:CodedValue</a>]</small>
</td><td width="50%"><a href="#CodeProvenance_RDF" title=""> CodeProvenance</a>
</td></tr>
</tbody></table>

##DataType RDF

DataType is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF)

##Name RDF

Name is a subtype of and inherits properties from [Component](http://wiki.chip.org/smart-project/index.php/Developers_Documentation:_SMART_Data_Model#Component_RDF)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://www.w3.org/2006/vcard/ns#Name" class="external free" title="http://www.w3.org/2006/vcard/ns#Name" rel="nofollow">http://www.w3.org/2006/vcard/ns#Name</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">additional-name<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Name" class="external text" title="http://www.w3.org/2006/vcard/ns#Name" rel="nofollow">v:Name</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>family-name</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Name" class="external text" title="http://www.w3.org/2006/vcard/ns#Name" rel="nofollow">v:Name</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>given-name</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Name" class="external text" title="http://www.w3.org/2006/vcard/ns#Name" rel="nofollow">v:Name</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">honorific-prefix<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Name" class="external text" title="http://www.w3.org/2006/vcard/ns#Name" rel="nofollow">v:Name</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">honorific-suffix<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Name" class="external text" title="http://www.w3.org/2006/vcard/ns#Name" rel="nofollow">v:Name</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##NarrativeResult RDF

NarrativeResult is a subtype of and inherits properties from Component

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#NarrativeResult" class="external free" title="http://smartplatforms.org/terms#NarrativeResult" rel="nofollow">http://smartplatforms.org/terms#NarrativeResult</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>value</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#NarrativeResult" class="external text" title="http://smartplatforms.org/terms#NarrativeResult" rel="nofollow">sp:NarrativeResult</a>]</small>
</td><td width="50%">Value of result (free text) [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##NominalResult RDF

NominalResult is a subtype of and inherits properties from Component 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#NominalResult" class="external free" title="http://smartplatforms.org/terms#NominalResult" rel="nofollow">http://smartplatforms.org/terms#NominalResult</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>value</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#NominalResult" class="external text" title="http://smartplatforms.org/terms#NominalResult" rel="nofollow">sp:NominalResult</a>]</small>
</td><td width="50%">value of result (free text). [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##OrdinalResult RDF

OrdinalResult is a subtype of and inherits properties from Component 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#OrdinalResult" class="external free" title="http://smartplatforms.org/terms#OrdinalResult" rel="nofollow">http://smartplatforms.org/terms#OrdinalResult</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>value</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#OrdinalResult" class="external text" title="http://smartplatforms.org/terms#OrdinalResult" rel="nofollow">sp:OrdinalResult</a>]</small>
</td><td width="50%">value of result (free text). [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##Organization RDF

Organization is a subtype of and inherits properties from Component

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Organization" class="external free" title="http://smartplatforms.org/terms#Organization" rel="nofollow">http://smartplatforms.org/terms#Organization</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">adr<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Organization" class="external text" title="http://smartplatforms.org/terms#Organization" rel="nofollow">sp:Organization</a>]</small>
</td><td width="50%"><a href="#Address_RDF" title=""> Address</a>
</td></tr>
<tr>
<td width="30%">organization-name<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Organization" class="external text" title="http://smartplatforms.org/terms#Organization" rel="nofollow">sp:Organization</a>]</small>
</td><td width="50%">Name of the organization [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##Participant RDF

Participant is a subtype of and inherits properties from Component

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Participant" class="external free" title="http://smartplatforms.org/terms#Participant" rel="nofollow">http://smartplatforms.org/terms#Participant</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">organization<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Participant" class="external text" title="http://smartplatforms.org/terms#Participant" rel="nofollow">sp:Participant</a>]</small>
</td><td width="50%"><a href="#Organization_RDF" title=""> Organization</a>
<p>Organization of participant
</p>
</td></tr>
<tr>
<td width="30%">person<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Participant" class="external text" title="http://smartplatforms.org/terms#Participant" rel="nofollow">sp:Participant</a>]</small>
</td><td width="50%"><a href="#Person_RDF" title=""> Person</a>
<p>Person who participated
</p>
</td></tr>
<tr>
<td width="30%">role<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Participant" class="external text" title="http://smartplatforms.org/terms#Participant" rel="nofollow">sp:Participant</a>]</small>
</td><td width="50%">Role of participant (free text) [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##Person RDF

Person is a subtype of and inherits properties from Component, VCard

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Person" class="external free" title="http://smartplatforms.org/terms#Person" rel="nofollow">http://smartplatforms.org/terms#Person</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">ethnicity<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">preferredLanguage<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">race<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">adr<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"><a href="#Address_RDF" title=""> Address</a>
</td></tr>
<tr>
<td width="30%">bday<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">email<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>n</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"><a href="#Name_RDF" title=""> Name</a>
</td></tr>
<tr>
<td width="30%">tel<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%"><a href="#Tel_RDF" title=""> Tel</a>
</td></tr>
<tr>
<td width="30%">gender<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Person" class="external text" title="http://smartplatforms.org/terms#Person" rel="nofollow">sp:Person</a>]</small>
</td><td width="50%">A person's (administrative) gender.  This should consist of the string "male" or "female". [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##Pharmacy RDF

Pharmacy is a subtype of and inherits properties from Component, Organization


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Pharmacy" class="external free" title="http://smartplatforms.org/terms#Pharmacy" rel="nofollow">http://smartplatforms.org/terms#Pharmacy</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">ncpdpId<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Pharmacy" class="external text" title="http://smartplatforms.org/terms#Pharmacy" rel="nofollow">sp:Pharmacy</a>]</small>
</td><td width="50%">Pharmacy's National Council for Prescription Drug Programs ID Number (NCPDP ID) [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">adr<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Pharmacy" class="external text" title="http://smartplatforms.org/terms#Pharmacy" rel="nofollow">sp:Pharmacy</a>]</small>
</td><td width="50%"><a href="#Address_RDF" title=""> Address</a>
</td></tr>
<tr>
<td width="30%">organization-name<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Pharmacy" class="external text" title="http://smartplatforms.org/terms#Pharmacy" rel="nofollow">sp:Pharmacy</a>]</small>
</td><td width="50%">Name of the organization [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##Provider RDF

Provider is a subtype of and inherits properties from Component, Person, VCard

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#Provider" class="external free" title="http://smartplatforms.org/terms#Provider" rel="nofollow">http://smartplatforms.org/terms#Provider</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">deaNumber<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%">Provider's Drug Enforcement Agency Number [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">ethnicity<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">npiNumber<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%">Provider's National Provider Identification Number [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">preferredLanguage<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">race<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">adr<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"><a href="#Address_RDF" title=""> Address</a>
</td></tr>
<tr>
<td width="30%">bday<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">email<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>n</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"><a href="#Name_RDF" title=""> Name</a>
</td></tr>
<tr>
<td width="30%">tel<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%"><a href="#Tel_RDF" title=""> Tel</a>
</td></tr>
<tr>
<td width="30%">gender<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#Provider" class="external text" title="http://smartplatforms.org/terms#Provider" rel="nofollow">sp:Provider</a>]</small>
</td><td width="50%">A person's (administrative) gender.  This should consist of the string "male" or "female". [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##QuantitativeResult RDF

QuantitativeResult is a subtype of and inherits properties from Component 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#QuantitativeResult" class="external free" title="http://smartplatforms.org/terms#QuantitativeResult" rel="nofollow">http://smartplatforms.org/terms#QuantitativeResult</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">nonCriticalRange<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#QuantitativeResult" class="external text" title="http://smartplatforms.org/terms#QuantitativeResult" rel="nofollow">sp:QuantitativeResult</a>]</small>
</td><td width="50%"><a href="#ValueRange_RDF" title=""> ValueRange</a>
<p>Non-critical range for result.  (Results outside this range are considered "critical.")
</p>
</td></tr>
<tr>
<td width="30%">normalRange<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#QuantitativeResult" class="external text" title="http://smartplatforms.org/terms#QuantitativeResult" rel="nofollow">sp:QuantitativeResult</a>]</small>
</td><td width="50%"><a href="#ValueRange_RDF" title=""> ValueRange</a>
<p>Normal range for result. (Results outside this range are considered "abnormal".)
</p>
</td></tr>
<tr>
<td width="30%"><b>valueAndUnit</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#QuantitativeResult" class="external text" title="http://smartplatforms.org/terms#QuantitativeResult" rel="nofollow">sp:QuantitativeResult</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p>Value and unit
</p>
</td></tr>
</tbody></table>

##Tel RDF

Tel is a subtype of and inherits properties from Component 


A v Tel element can be given additional types to indicate 

* Preferred Status (v:Pref)
* Cell phone (vCell)
* Home phone (vHome)
* Work phone (vWork)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://www.w3.org/2006/vcard/ns#Tel" class="external free" title="http://www.w3.org/2006/vcard/ns#Tel" rel="nofollow">http://www.w3.org/2006/vcard/ns#Tel</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>value</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#Tel" class="external text" title="http://www.w3.org/2006/vcard/ns#Tel" rel="nofollow">v:Tel</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##VCard RDF

VCard is a subtype of and inherits properties from Component 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://www.w3.org/2006/vcard/ns#VCard" class="external free" title="http://www.w3.org/2006/vcard/ns#VCard" rel="nofollow">http://www.w3.org/2006/vcard/ns#VCard</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">adr<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#VCard" class="external text" title="http://www.w3.org/2006/vcard/ns#VCard" rel="nofollow">v:VCard</a>]</small>
</td><td width="50%"><a href="#Address_RDF" title=""> Address</a>
</td></tr>
<tr>
<td width="30%">bday<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#VCard" class="external text" title="http://www.w3.org/2006/vcard/ns#VCard" rel="nofollow">v:VCard</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">email<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#VCard" class="external text" title="http://www.w3.org/2006/vcard/ns#VCard" rel="nofollow">v:VCard</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>n</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#VCard" class="external text" title="http://www.w3.org/2006/vcard/ns#VCard" rel="nofollow">v:VCard</a>]</small>
</td><td width="50%"><a href="#Name_RDF" title=""> Name</a>
</td></tr>
<tr>
<td width="30%">tel<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://www.w3.org/2006/vcard/ns#VCard" class="external text" title="http://www.w3.org/2006/vcard/ns#VCard" rel="nofollow">v:VCard</a>]</small>
</td><td width="50%"><a href="#Tel_RDF" title=""> Tel</a>
</td></tr>
</tbody></table>

##ValueAndUnit RDF

ValueAndUnit is a subtype of and inherits properties from Component, DataType 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#ValueAndUnit" class="external free" title="http://smartplatforms.org/terms#ValueAndUnit" rel="nofollow">http://smartplatforms.org/terms#ValueAndUnit</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>unit</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#ValueAndUnit" class="external text" title="http://smartplatforms.org/terms#ValueAndUnit" rel="nofollow">sp:ValueAndUnit</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>value</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#ValueAndUnit" class="external text" title="http://smartplatforms.org/terms#ValueAndUnit" rel="nofollow">sp:ValueAndUnit</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##ValueRange RDF

ValueRange is a subtype of and inherits properties from Component, DataType 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#ValueRange" class="external free" title="http://smartplatforms.org/terms#ValueRange" rel="nofollow">http://smartplatforms.org/terms#ValueRange</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>maximum</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#ValueRange" class="external text" title="http://smartplatforms.org/terms#ValueRange" rel="nofollow">sp:ValueRange</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p>Maximum value in range (not inclusive)
</p>
</td></tr>
<tr>
<td width="30%"><b>minimum</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#ValueRange" class="external text" title="http://smartplatforms.org/terms#ValueRange" rel="nofollow">sp:ValueRange</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p>Minimum value in range (inclusive)
</p>
</td></tr>
</tbody></table>

##ValueRatio RDF

ValueRatio is a subtype of and inherits properties from Component, DataType 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#ValueRatio" class="external free" title="http://smartplatforms.org/terms#ValueRatio" rel="nofollow">http://smartplatforms.org/terms#ValueRatio</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>denominator</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#ValueRatio" class="external text" title="http://smartplatforms.org/terms#ValueRatio" rel="nofollow">sp:ValueRatio</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p>Denominator of the ratio.
</p>
</td></tr>
<tr>
<td width="30%"><b>numerator</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#ValueRatio" class="external text" title="http://smartplatforms.org/terms#ValueRatio" rel="nofollow">sp:ValueRatio</a>]</small>
</td><td width="50%"><a href="#ValueAndUnit_RDF" title=""> ValueAndUnit</a>
<p>Numerator of the ratio.
</p>
</td></tr>
</tbody></table>

##VitalSign RDF

VitalSign is a subtype of and inherits properties from Component, DataType, ValueAndUnit 

Vital Sign: includes a LOINC code specifying which measurement is being reported, alongside a value and unit. 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#VitalSign" class="external free" title="http://smartplatforms.org/terms#VitalSign" rel="nofollow">http://smartplatforms.org/terms#VitalSign</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>unit</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSign" class="external text" title="http://smartplatforms.org/terms#VitalSign" rel="nofollow">sp:VitalSign</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>value</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSign" class="external text" title="http://smartplatforms.org/terms#VitalSign" rel="nofollow">sp:VitalSign</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">vitalName<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#VitalSign" class="external text" title="http://smartplatforms.org/terms#VitalSign" rel="nofollow">sp:VitalSign</a>]</small>
</td><td width="50%"><a href="#Coded_Value_RDF" title=""> Coded Value</a> where code comes from <a href="#VitalSign_code_RDF" title=""> VitalSign</a>
<p>LOINC Coded Value for the vital sign type
</p>
</td></tr>
</tbody></table>

#Container-level Types

##App Manifest RDF

##Capabilities RDF

A SMART Container exposes a set of capabilities as a JSON structure. The example below shows the capabilities of a container that provides Demographics, Encounters, and Vital Signs only. 

{% highlight html %}
{
    "http://smartplatforms.org/terms#Demographics": {
        "methods": [
            "GET"
        ]
    }, 
    "http://smartplatforms.org/terms#Encounter": {
        "methods": [
            "GET"
        ]
    }, 
    "http://smartplatforms.org/terms#VitalSigns": {
        "methods": [
            "GET"
        ]
    }
}
{% endhighlight  %}

##Ontology RDF

User is a subtype of and inherits properties from Component, Person, VCard 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms#User" class="external free" title="http://smartplatforms.org/terms#User" rel="nofollow">http://smartplatforms.org/terms#User</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%">department<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%">A user's department [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">ethnicity<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">preferredLanguage<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">race<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">role<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%">A user's role [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">adr<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"><a href="#Address_RDF" title=""> Address</a>
</td></tr>
<tr>
<td width="30%">bday<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%">email<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>n</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"><a href="#Name_RDF" title=""> Name</a>
</td></tr>
<tr>
<td width="30%">tel<br><small>Optional: 0 or more</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%"><a href="#Tel_RDF" title=""> Tel</a>
</td></tr>
<tr>
<td width="30%">gender<br><small>Optional: 0 or 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms#User" class="external text" title="http://smartplatforms.org/terms#User" rel="nofollow">sp:User</a>]</small>
</td><td width="50%">A person's (administrative) gender.  This should consist of the string "male" or "female". [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

#Data code Types

##AlertLevel code RDF

AlertLevel is a subtype of and inherits properties from Code, Component, DataType 

Constrained to one of 

{% highlight html %}
<spcode:AlertLevel rdf:about="http://smartplatforms.org/terms/codes/AlertLevel#warning">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Warning</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/AlertLevel#</sp:system>
    <dcterms:identifier>warning</dcterms:identifier> 
</spcode:AlertLevel>
<spcode:AlertLevel rdf:about="http://smartplatforms.org/terms/codes/AlertLevel#severe">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Severe</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/AlertLevel#</sp:system>
    <dcterms:identifier>severe</dcterms:identifier> 
</spcode:AlertLevel>
<spcode:AlertLevel rdf:about="http://smartplatforms.org/terms/codes/AlertLevel#info">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Info</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/AlertLevel#</sp:system>
    <dcterms:identifier>info</dcterms:identifier> 
</spcode:AlertLevel>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/AlertLevel" class="external free" title="http://smartplatforms.org/terms/codes/AlertLevel" rel="nofollow">http://smartplatforms.org/terms/codes/AlertLevel</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AlertLevel" class="external text" title="http://smartplatforms.org/terms/codes/AlertLevel" rel="nofollow">spcode:AlertLevel</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AlertLevel" class="external text" title="http://smartplatforms.org/terms/codes/AlertLevel" rel="nofollow">spcode:AlertLevel</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AlertLevel" class="external text" title="http://smartplatforms.org/terms/codes/AlertLevel" rel="nofollow">spcode:AlertLevel</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##AllergyCategory code RDF

AllergyCategory is a subtype of and inherits properties from Code, Component, DataType, SNOMED 

Constrained to one of

{% highlight html %}
<spcode:AllergyCategory rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/414285001">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Food allergy</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>414285001</dcterms:identifier> 
</spcode:AllergyCategory>
<spcode:AllergyCategory rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/426232007">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Environmental allergy</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>426232007</dcterms:identifier> 
</spcode:AllergyCategory>
<spcode:AllergyCategory rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/416098002">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Drug allergy</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>416098002</dcterms:identifier> 
</spcode:AllergyCategory>
<spcode:AllergyCategory rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/59037007">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Drug intolerance</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>59037007</dcterms:identifier> 
</spcode:AllergyCategory>
<spcode:AllergyCategory rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/235719002">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Food intolerance</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>235719002</dcterms:identifier> 
</spcode:AllergyCategory>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/AllergyCategory" class="external free" title="http://smartplatforms.org/terms/codes/AllergyCategory" rel="nofollow">http://smartplatforms.org/terms/codes/AllergyCategory</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergyCategory" class="external text" title="http://smartplatforms.org/terms/codes/AllergyCategory" rel="nofollow">spcode:AllergyCategory</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergyCategory" class="external text" title="http://smartplatforms.org/terms/codes/AllergyCategory" rel="nofollow">spcode:AllergyCategory</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergyCategory" class="external text" title="http://smartplatforms.org/terms/codes/AllergyCategory" rel="nofollow">spcode:AllergyCategory</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##AllergyExclusion code RDF

AllergyExclusion is a subtype of and inherits properties from Code, Component, DataType, SNOMED 

Constrained to one of

{% highlight html %}
<spcode:AllergyExclusion rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/160244002">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>No known allergies</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>160244002</dcterms:identifier> 
</spcode:AllergyExclusion>
<spcode:AllergyExclusion rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/428607008">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>No known environmental allergy</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>428607008</dcterms:identifier> 
</spcode:AllergyExclusion>
<spcode:AllergyExclusion rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/429625007">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>No known food allergy</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>429625007</dcterms:identifier> 
</spcode:AllergyExclusion>
<spcode:AllergyExclusion rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/409137002">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>No known history of drug allergy</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>409137002</dcterms:identifier> 
</spcode:AllergyExclusion>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/AllergyExclusion" class="external free" title="http://smartplatforms.org/terms/codes/AllergyExclusion" rel="nofollow">http://smartplatforms.org/terms/codes/AllergyExclusion</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergyExclusion" class="external text" title="http://smartplatforms.org/terms/codes/AllergyExclusion" rel="nofollow">spcode:AllergyExclusion</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergyExclusion" class="external text" title="http://smartplatforms.org/terms/codes/AllergyExclusion" rel="nofollow">spcode:AllergyExclusion</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergyExclusion" class="external text" title="http://smartplatforms.org/terms/codes/AllergyExclusion" rel="nofollow">spcode:AllergyExclusion</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##AllergySeverity code RDF

AllergySeverity is a subtype of and inherits properties from Code, Component, DataType, SNOMED 


Constrained to one of 

{% highlight html %}
<spcode:AllergySeverity rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/255604002">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Mild</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>255604002</dcterms:identifier> 
</spcode:AllergySeverity>
<spcode:AllergySeverity rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/442452003">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Life threatening</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>442452003</dcterms:identifier> 
</spcode:AllergySeverity>
<spcode:AllergySeverity rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/6736007">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Moderate</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>6736007</dcterms:identifier> 
</spcode:AllergySeverity>
<spcode:AllergySeverity rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/399166001">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Fatal</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>399166001</dcterms:identifier> 
</spcode:AllergySeverity>
<spcode:AllergySeverity rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/24484000">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Severe</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>24484000</dcterms:identifier> 
</spcode:AllergySeverity>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/AllergySeverity" class="external free" title="http://smartplatforms.org/terms/codes/AllergySeverity" rel="nofollow">http://smartplatforms.org/terms/codes/AllergySeverity</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergySeverity" class="external text" title="http://smartplatforms.org/terms/codes/AllergySeverity" rel="nofollow">spcode:AllergySeverity</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergySeverity" class="external text" title="http://smartplatforms.org/terms/codes/AllergySeverity" rel="nofollow">spcode:AllergySeverity</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/AllergySeverity" class="external text" title="http://smartplatforms.org/terms/codes/AllergySeverity" rel="nofollow">spcode:AllergySeverity</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##BloodPressureBodyPosition code RDF

BloodPressureBodyPosition is a subtype of and inherits properties from Code, Component, DataType, SNOMED 

Constrained to one of 

{% highlight html %}
<spcode:BloodPressureBodyPosition rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/40199007">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Supine</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>40199007</dcterms:identifier> 
</spcode:BloodPressureBodyPosition>
<spcode:BloodPressureBodyPosition rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/33586001">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Sitting</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>33586001</dcterms:identifier> 
</spcode:BloodPressureBodyPosition>
<spcode:BloodPressureBodyPosition rdf:about="http://purl.bioontology.org/ontology/SNOMEDCT/10904000">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Standing</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/SNOMEDCT/</sp:system>
    <dcterms:identifier>10904000</dcterms:identifier> 
</spcode:BloodPressureBodyPosition>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" class="external free" title="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" rel="nofollow">http://smartplatforms.org/terms/codes/BloodPressureBodyPosition</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" class="external text" title="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" rel="nofollow">spcode:BloodPressureBodyPosition</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" class="external text" title="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" rel="nofollow">spcode:BloodPressureBodyPosition</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" class="external text" title="http://smartplatforms.org/terms/codes/BloodPressureBodyPosition" rel="nofollow">spcode:BloodPressureBodyPosition</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##BloodPressureBodySite code RDF

BloodPressureBodySite is a subtype of and inherits properties from Code, Component, DataType, SNOMED 

Constrained to one of

{% highlight html %}
<spcode:BloodPressureMethod rdf:about="http://smartplatforms.org/terms/codes/BloodPressureMethod#invasive">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Invasive</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/BloodPressureMethod#</sp:system>
    <dcterms:identifier>invasive</dcterms:identifier> 
</spcode:BloodPressureMethod>
<spcode:BloodPressureMethod rdf:about="http://smartplatforms.org/terms/codes/BloodPressureMethod#palpation">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Palpation</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/BloodPressureMethod#</sp:system>
    <dcterms:identifier>palpation</dcterms:identifier> 
</spcode:BloodPressureMethod>
<spcode:BloodPressureMethod rdf:about="http://smartplatforms.org/terms/codes/BloodPressureMethod#machine">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Machine</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/BloodPressureMethod#</sp:system>
    <dcterms:identifier>machine</dcterms:identifier> 
</spcode:BloodPressureMethod>
<spcode:BloodPressureMethod rdf:about="http://smartplatforms.org/terms/codes/BloodPressureMethod#auscultation">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Auscultation</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/BloodPressureMethod#</sp:system>
    <dcterms:identifier>auscultation</dcterms:identifier> 
</spcode:BloodPressureMethod>
{% endhighlight  %}


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/BloodPressureMethod" class="external free" title="http://smartplatforms.org/terms/codes/BloodPressureMethod" rel="nofollow">http://smartplatforms.org/terms/codes/BloodPressureMethod</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/BloodPressureMethod" class="external text" title="http://smartplatforms.org/terms/codes/BloodPressureMethod" rel="nofollow">spcode:BloodPressureMethod</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/BloodPressureMethod" class="external text" title="http://smartplatforms.org/terms/codes/BloodPressureMethod" rel="nofollow">spcode:BloodPressureMethod</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/BloodPressureMethod" class="external text" title="http://smartplatforms.org/terms/codes/BloodPressureMethod" rel="nofollow">spcode:BloodPressureMethod</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##EncounterType code RDF

EncounterType is a subtype of and inherits properties from Code, Component, DataType 

Constrained to one of 

{% highlight html %}
<spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#home">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Home encounter</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
    <dcterms:identifier>home</dcterms:identifier> 
</spcode:EncounterType>
<spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#emergency">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Emergency encounter</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
    <dcterms:identifier>emergency</dcterms:identifier> 
</spcode:EncounterType>
<spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#ambulatory">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Ambulatory encounter</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
    <dcterms:identifier>ambulatory</dcterms:identifier> 
</spcode:EncounterType>
<spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#inpatient">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Inpatient encounter</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
    <dcterms:identifier>inpatient</dcterms:identifier> 
</spcode:EncounterType>
<spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#field">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Field encounter</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
    <dcterms:identifier>field</dcterms:identifier> 
</spcode:EncounterType>
<spcode:EncounterType rdf:about="http://smartplatforms.org/terms/codes/EncounterType#virtual">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Virtual encounter</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/EncounterType#</sp:system>
    <dcterms:identifier>virtual</dcterms:identifier> 
</spcode:EncounterType>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/EncounterType" class="external free" title="http://smartplatforms.org/terms/codes/EncounterType" rel="nofollow">http://smartplatforms.org/terms/codes/EncounterType</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/EncounterType" class="external text" title="http://smartplatforms.org/terms/codes/EncounterType" rel="nofollow">spcode:EncounterType</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/EncounterType" class="external text" title="http://smartplatforms.org/terms/codes/EncounterType" rel="nofollow">spcode:EncounterType</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/EncounterType" class="external text" title="http://smartplatforms.org/terms/codes/EncounterType" rel="nofollow">spcode:EncounterType</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##ImmunizationAdministrationStatus code RDF

ImmunizationAdministrationStatus is a subtype of and inherits properties from Code, Component, DataType 

Constrained to one of 

{% highlight html %}
<spcode:ImmunizationAdministrationStatus rdf:about="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#doseGiven">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Dose Given</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#</sp:system>
    <dcterms:identifier>doseGiven</dcterms:identifier> 
</spcode:ImmunizationAdministrationStatus>
<spcode:ImmunizationAdministrationStatus rdf:about="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#notAdministered">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Not Administered</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#</sp:system>
    <dcterms:identifier>notAdministered</dcterms:identifier> 
</spcode:ImmunizationAdministrationStatus>
<spcode:ImmunizationAdministrationStatus rdf:about="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#partialDose">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Dose Partially Administered</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus#</sp:system>
    <dcterms:identifier>partialDose</dcterms:identifier> 
</spcode:ImmunizationAdministrationStatus>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" class="external free" title="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" rel="nofollow">http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" rel="nofollow">spcode:ImmunizationAdministrationStatus</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" rel="nofollow">spcode:ImmunizationAdministrationStatus</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationAdministrationStatus" rel="nofollow">spcode:ImmunizationAdministrationStatus</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##ImmunizationClass code RDF

ImmunizationClass is a subtype of and inherits properties from Code, Component, DataType 

codes are drawn from the CDC's Vaccine Group vocabulary. URIs are of the form

[http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#code](http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#code)

system = [http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#](http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/ImmunizationClass" class="external free" title="http://smartplatforms.org/terms/codes/ImmunizationClass" rel="nofollow">http://smartplatforms.org/terms/codes/ImmunizationClass</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationClass" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationClass" rel="nofollow">spcode:ImmunizationClass</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationClass" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationClass" rel="nofollow">spcode:ImmunizationClass</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationClass" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationClass" rel="nofollow">spcode:ImmunizationClass</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##ImmunizationProduct code RDF

ImmunizationProduct is a subtype of and inherits properties from Code, Component, DataType

codes are drawn from the CDC's Vaccine Group vocabulary. URIs are of the form 

[http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#code ](http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#code)


system = [http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#](http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#)

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/ImmunizationProduct" class="external free" title="http://smartplatforms.org/terms/codes/ImmunizationProduct" rel="nofollow">http://smartplatforms.org/terms/codes/ImmunizationProduct</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationProduct" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationProduct" rel="nofollow">spcode:ImmunizationProduct</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationProduct" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationProduct" rel="nofollow">spcode:ImmunizationProduct</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationProduct" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationProduct" rel="nofollow">spcode:ImmunizationProduct</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##ImmunizationRefusalReason code RDF

ImmunizationRefusalReason is a subtype of and inherits properties from Code, Component, DataType 


Constrained to one of 
{% highlight html %}
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#vaccineUnavailable">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Vaccine unavailable at visit</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>vaccineUnavailable</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#patientUndergoingDesensitizationTherapy">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Patient undergoing desensitization therapy</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>patientUndergoingDesensitizationTherapy</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#notIndicatedPerGuidelines">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Not indicated per guidelines</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>notIndicatedPerGuidelines</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#recentChemoOrRadiaton">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Recent chemotherapy/radiaton</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>recentChemoOrRadiaton</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#allergy">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Allergy to vaccine/vaccine components, or allergy to eggs</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>allergy</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#providerDeferred">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Provider deferred</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>providerDeferred</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#documentedImmunityOrPreviousDisease">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Documented immunity or previous disease</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>documentedImmunityOrPreviousDisease</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#previouslyVaccinated">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Previously vaccinated</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>previouslyVaccinated</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#contraindicated">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Contraindicated</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>contraindicated</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#patientOrParentRefused">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Patient/parent refused</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>patientOrParentRefused</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#comfortMeasuresOnly">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Comfort Measures Only</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>comfortMeasuresOnly</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#possiblePriorAllergyOrReaction">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Possible allergy/reaction to prior dose</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>possiblePriorAllergyOrReaction</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
<spcode:ImmunizationRefusalReason rdf:about="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#recentOrganOrStemCellTransplant">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Recent organ/stem cell transplant</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/ImmunizationRefusalReason#</sp:system>
    <dcterms:identifier>recentOrganOrStemCellTransplant</dcterms:identifier> 
</spcode:ImmunizationRefusalReason>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" class="external free" title="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" rel="nofollow">http://smartplatforms.org/terms/codes/ImmunizationRefusalReason</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" rel="nofollow">spcode:ImmunizationRefusalReason</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" rel="nofollow">spcode:ImmunizationRefusalReason</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" class="external text" title="http://smartplatforms.org/terms/codes/ImmunizationRefusalReason" rel="nofollow">spcode:ImmunizationRefusalReason</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##LOINC code RDF

LOINC is a subtype of and inherits properties from Code, Component, DataType 


LOINC code where URI matches [http://purl.bioontology.org/ontology/LNC/](http://purl.bioontology.org/ontology/LNC/){loinc_code} 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/LOINC" class="external free" title="http://smartplatforms.org/terms/codes/LOINC" rel="nofollow">http://smartplatforms.org/terms/codes/LOINC</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LOINC" class="external text" title="http://smartplatforms.org/terms/codes/LOINC" rel="nofollow">spcode:LOINC</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LOINC" class="external text" title="http://smartplatforms.org/terms/codes/LOINC" rel="nofollow">spcode:LOINC</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LOINC" class="external text" title="http://smartplatforms.org/terms/codes/LOINC" rel="nofollow">spcode:LOINC</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##LabResultInterpretation code RDF

LabResultInterpretation is a subtype of and inherits properties from Code, Component, DataType

Constrained to one of 

{% highlight html %}
<spcode:LabResultInterpretation rdf:about="http://smartplatforms.org/terms/codes/LabResultInterpretation#normal">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Normal</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/LabResultInterpretation#</sp:system>
    <dcterms:identifier>normal</dcterms:identifier> 
</spcode:LabResultInterpretation>
<spcode:LabResultInterpretation rdf:about="http://smartplatforms.org/terms/codes/LabResultInterpretation#critical">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Critical</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/LabResultInterpretation#</sp:system>
    <dcterms:identifier>critical</dcterms:identifier> 
</spcode:LabResultInterpretation>
<spcode:LabResultInterpretation rdf:about="http://smartplatforms.org/terms/codes/LabResultInterpretation#abnormal">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Abnormal</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/LabResultInterpretation#</sp:system>
    <dcterms:identifier>abnormal</dcterms:identifier> 
</spcode:LabResultInterpretation>
{% endhighlight  %}


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/LabResultInterpretation" class="external free" title="http://smartplatforms.org/terms/codes/LabResultInterpretation" rel="nofollow">http://smartplatforms.org/terms/codes/LabResultInterpretation</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LabResultInterpretation" class="external text" title="http://smartplatforms.org/terms/codes/LabResultInterpretation" rel="nofollow">spcode:LabResultInterpretation</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LabResultInterpretation" class="external text" title="http://smartplatforms.org/terms/codes/LabResultInterpretation" rel="nofollow">spcode:LabResultInterpretation</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LabResultInterpretation" class="external text" title="http://smartplatforms.org/terms/codes/LabResultInterpretation" rel="nofollow">spcode:LabResultInterpretation</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##LabResultStatus code RDF

LabResultStatus is a subtype of and inherits properties from Code, Component, DataType 

Constrained to one of 

{% highlight html %}
<spcode:LabResultStatus rdf:about="http://smartplatforms.org/terms/codes/LabStatus#correction">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Correction</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/LabStatus#</sp:system>
    <dcterms:identifier>correction</dcterms:identifier> 
</spcode:LabResultStatus>
<spcode:LabResultStatus rdf:about="http://smartplatforms.org/terms/codes/LabStatus#preliminary">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Preliminary</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/LabStatus#</sp:system>
    <dcterms:identifier>preliminary</dcterms:identifier> 
</spcode:LabResultStatus>
<spcode:LabResultStatus rdf:about="http://smartplatforms.org/terms/codes/LabStatus#final">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Final</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/LabStatus#</sp:system>
    <dcterms:identifier>final</dcterms:identifier> 
</spcode:LabResultStatus>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/LabResultStatus" class="external free" title="http://smartplatforms.org/terms/codes/LabResultStatus" rel="nofollow">http://smartplatforms.org/terms/codes/LabResultStatus</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LabResultStatus" class="external text" title="http://smartplatforms.org/terms/codes/LabResultStatus" rel="nofollow">spcode:LabResultStatus</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LabResultStatus" class="external text" title="http://smartplatforms.org/terms/codes/LabResultStatus" rel="nofollow">spcode:LabResultStatus</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/LabResultStatus" class="external text" title="http://smartplatforms.org/terms/codes/LabResultStatus" rel="nofollow">spcode:LabResultStatus</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##MedicalRecordNumber code RDF

MedicalRecordNumber is a subtype of and inherits properties from Code, Component, DataType 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/MedicalRecordNumber" class="external free" title="http://smartplatforms.org/terms/codes/MedicalRecordNumber" rel="nofollow">http://smartplatforms.org/terms/codes/MedicalRecordNumber</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/MedicalRecordNumber" class="external text" title="http://smartplatforms.org/terms/codes/MedicalRecordNumber" rel="nofollow">spcode:MedicalRecordNumber</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/MedicalRecordNumber" class="external text" title="http://smartplatforms.org/terms/codes/MedicalRecordNumber" rel="nofollow">spcode:MedicalRecordNumber</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/MedicalRecordNumber" class="external text" title="http://smartplatforms.org/terms/codes/MedicalRecordNumber" rel="nofollow">spcode:MedicalRecordNumber</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##MedicationProvenance code RDF

MedicationProvenance is a subtype of and inherits properties from Code, Component, DataType 

Constrained to one of 
{% highlight html %}
<spcode:MedicationProvenance rdf:about="http://smartplatforms.org/terms/codes/MedicationProvenance#prescription">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Derived by prescription</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/MedicationProvenance#</sp:system>
    <dcterms:identifier>prescription</dcterms:identifier> 
</spcode:MedicationProvenance>
<spcode:MedicationProvenance rdf:about="http://smartplatforms.org/terms/codes/MedicationProvenance#fulfillment">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Derived by fulfillment</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/MedicationProvenance#</sp:system>
    <dcterms:identifier>fulfillment</dcterms:identifier> 
</spcode:MedicationProvenance>
<spcode:MedicationProvenance rdf:about="http://smartplatforms.org/terms/codes/MedicationProvenance#administration">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Derived by administration</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/MedicationProvenance#</sp:system>
    <dcterms:identifier>administration</dcterms:identifier> 
</spcode:MedicationProvenance>
<spcode:MedicationProvenance rdf:about="http://smartplatforms.org/terms/codes/MedicationProvenance#reconciliation">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Derived by medication reconciliation</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/MedicationProvenance#</sp:system>
    <dcterms:identifier>reconciliation</dcterms:identifier> 
</spcode:MedicationProvenance>
<spcode:MedicationProvenance rdf:about="http://smartplatforms.org/terms/codes/MedicationProvenance#patientReport">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Derived by patient report</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/MedicationProvenance#</sp:system>
    <dcterms:identifier>patientReport</dcterms:identifier> 
</spcode:MedicationProvenance>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/MedicationProvenance" class="external free" title="http://smartplatforms.org/terms/codes/MedicationProvenance" rel="nofollow">http://smartplatforms.org/terms/codes/MedicationProvenance</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/MedicationProvenance" class="external text" title="http://smartplatforms.org/terms/codes/MedicationProvenance" rel="nofollow">spcode:MedicationProvenance</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/MedicationProvenance" class="external text" title="http://smartplatforms.org/terms/codes/MedicationProvenance" rel="nofollow">spcode:MedicationProvenance</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/MedicationProvenance" class="external text" title="http://smartplatforms.org/terms/codes/MedicationProvenance" rel="nofollow">spcode:MedicationProvenance</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##NDFRT code RDF

NDFRT is a subtype of and inherits properties from Code, Component, DataType 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/NDFRT" class="external free" title="http://smartplatforms.org/terms/codes/NDFRT" rel="nofollow">http://smartplatforms.org/terms/codes/NDFRT</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/NDFRT" class="external text" title="http://smartplatforms.org/terms/codes/NDFRT" rel="nofollow">spcode:NDFRT</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/NDFRT" class="external text" title="http://smartplatforms.org/terms/codes/NDFRT" rel="nofollow">spcode:NDFRT</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/NDFRT" class="external text" title="http://smartplatforms.org/terms/codes/NDFRT" rel="nofollow">spcode:NDFRT</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>


##RxNorm code RDF

RxNorm is a subtype of and inherits properties from Code, Component, DataType 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/RxNorm" class="external free" title="http://smartplatforms.org/terms/codes/RxNorm" rel="nofollow">http://smartplatforms.org/terms/codes/RxNorm</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm" rel="nofollow">spcode:RxNorm</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm" rel="nofollow">spcode:RxNorm</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm" rel="nofollow">spcode:RxNorm</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##RxNorm_Ingredient code RDF

RxNorm_Ingredient is a subtype of and inherits properties from Code, Component, DataType, RxNorm 

RxNorm TTY='in' 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" class="external free" title="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" rel="nofollow">http://smartplatforms.org/terms/codes/RxNorm_Ingredient</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" rel="nofollow">spcode:RxNorm_Ingredient</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" rel="nofollow">spcode:RxNorm_Ingredient</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm_Ingredient" rel="nofollow">spcode:RxNorm_Ingredient</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##RxNorm_Semantic code RDF

xNorm_Semantic is a subtype of and inherits properties from Code, Component, DataType, RxNorm 


RxNorm TTY in ('SCD','SBD','GPCK','BPCK') 

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/RxNorm_Semantic" class="external free" title="http://smartplatforms.org/terms/codes/RxNorm_Semantic" rel="nofollow">http://smartplatforms.org/terms/codes/RxNorm_Semantic</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm_Semantic" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm_Semantic" rel="nofollow">spcode:RxNorm_Semantic</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm_Semantic" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm_Semantic" rel="nofollow">spcode:RxNorm_Semantic</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/RxNorm_Semantic" class="external text" title="http://smartplatforms.org/terms/codes/RxNorm_Semantic" rel="nofollow">spcode:RxNorm_Semantic</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##SNOMED code RDF

SNOMED is a subtype of and inherits properties from: Code, Component, DataType 


SNOMED code with URI matchign [http://purl.bioontology.org/ontology/SNOMEDCT/](http://purl.bioontology.org/ontology/SNOMEDCT/){snomed_concept_id} 
<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/SNOMED" class="external free" title="http://smartplatforms.org/terms/codes/SNOMED" rel="nofollow">http://smartplatforms.org/terms/codes/SNOMED</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/SNOMED" class="external text" title="http://smartplatforms.org/terms/codes/SNOMED" rel="nofollow">spcode:SNOMED</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/SNOMED" class="external text" title="http://smartplatforms.org/terms/codes/SNOMED" rel="nofollow">spcode:SNOMED</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/SNOMED" class="external text" title="http://smartplatforms.org/terms/codes/SNOMED" rel="nofollow">spcode:SNOMED</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##TranslationFidelity code RDF

TranslationFidelity is a subtype of and inherits properties from Code, Component, DataType 

Constrained to one of 
{% highlight html %}
<spcode:TranslationFidelity rdf:about="http://smartplatforms.org/terms/codes/TranslationFidelity#automated">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Automated</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/TranslationFidelity#</sp:system>
    <dcterms:identifier>automated</dcterms:identifier> 
</spcode:TranslationFidelity>
<spcode:TranslationFidelity rdf:about="http://smartplatforms.org/terms/codes/TranslationFidelity#unmappable">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Unmappable</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/TranslationFidelity#</sp:system>
    <dcterms:identifier>unmappable</dcterms:identifier> 
</spcode:TranslationFidelity>
<spcode:TranslationFidelity rdf:about="http://smartplatforms.org/terms/codes/TranslationFidelity#verified">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Verified</dcterms:title>
    <sp:system>http://smartplatforms.org/terms/codes/TranslationFidelity#</sp:system>
    <dcterms:identifier>verified</dcterms:identifier> 
</spcode:TranslationFidelity>

{% endhighlight  %}


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/TranslationFidelity" class="external free" title="http://smartplatforms.org/terms/codes/TranslationFidelity" rel="nofollow">http://smartplatforms.org/terms/codes/TranslationFidelity</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/TranslationFidelity" class="external text" title="http://smartplatforms.org/terms/codes/TranslationFidelity" rel="nofollow">spcode:TranslationFidelity</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/TranslationFidelity" class="external text" title="http://smartplatforms.org/terms/codes/TranslationFidelity" rel="nofollow">spcode:TranslationFidelity</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/TranslationFidelity" class="external text" title="http://smartplatforms.org/terms/codes/TranslationFidelity" rel="nofollow">spcode:TranslationFidelity</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##UNII code RDF

UNII is a subtype of and inherits properties from Code, Component, DataType 

UNII code with URI matching [http://fda.gov/UNII/{UNII}](http://fda.gov/UNII/)


<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/UNII" class="external free" title="http://smartplatforms.org/terms/codes/UNII" rel="nofollow">http://smartplatforms.org/terms/codes/UNII</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/UNII" class="external text" title="http://smartplatforms.org/terms/codes/UNII" rel="nofollow">spcode:UNII</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/UNII" class="external text" title="http://smartplatforms.org/terms/codes/UNII" rel="nofollow">spcode:UNII</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/UNII" class="external text" title="http://smartplatforms.org/terms/codes/UNII" rel="nofollow">spcode:UNII</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

##VitalSign code RDF

VitalSign is a subtype of and inherits properties from Code, Component, DataType, LOINC

Constrained to one of 

{% highlight html %}
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8462-4">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Intravascular diastolic</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>8462-4</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/9279-1">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Respiration rate</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>9279-1</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/39156-5">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Body mass index</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>39156-5</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8310-5">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Body temperature</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>8310-5</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/2710-2">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Oxygen saturation</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>2710-2</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8302-2">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Body height</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>8302-2</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8867-4">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Heart rate</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>8867-4</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8480-6">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Intravascular systolic</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>8480-6</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/3141-9">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Body weight</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>3141-9</dcterms:identifier> 
</spcode:VitalSign>
<spcode:VitalSign rdf:about="http://purl.bioontology.org/ontology/LNC/8306-3">
    <rdf:type rdf:resource="http://smartplatforms.org/terms#Code" /> 
    <dcterms:title>Body height (lying)</dcterms:title>
    <sp:system>http://purl.bioontology.org/ontology/LNC/</sp:system>
    <dcterms:identifier>8306-3</dcterms:identifier> 
</spcode:VitalSign>
{% endhighlight  %}

<table class="table table-striped">
<caption align="bottom"><i><a href="http://smartplatforms.org/terms/codes/VitalSign" class="external free" title="http://smartplatforms.org/terms/codes/VitalSign" rel="nofollow">http://smartplatforms.org/terms/codes/VitalSign</a> Properties</i>
</caption>
<tbody><tr>
<td width="30%"><b>identifier</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/VitalSign" class="external text" title="http://smartplatforms.org/terms/codes/VitalSign" rel="nofollow">spcode:VitalSign</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>title</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/VitalSign" class="external text" title="http://smartplatforms.org/terms/codes/VitalSign" rel="nofollow">spcode:VitalSign</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
<tr>
<td width="30%"><b>system</b><br><small>Required: exactly 1</small>
</td><td width="20%"><small>[<a href="http://smartplatforms.org/terms/codes/VitalSign" class="external text" title="http://smartplatforms.org/terms/codes/VitalSign" rel="nofollow">spcode:VitalSign</a>]</small>
</td><td width="50%"> [<a href="http://www.w3.org/2000/01/rdf-schema#Literal" class="external text" title="http://www.w3.org/2000/01/rdf-schema#Literal" rel="nofollow">rdfs:Literal</a>]
</td></tr>
</tbody></table>

