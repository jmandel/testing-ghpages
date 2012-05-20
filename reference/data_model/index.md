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


<table width="100%" border="0" cellspacing="0" class="table">
  <tr>
    <td>dcterms</td>
    <td>http://purl.org/dc/terms/</td>
    <td>Dublin core terms</td>
  </tr>
  
   <tr>
    <td>foaf</td>
    <td>http://xmlns.com/foaf/0.1/</td>
    <td>Friend of a friend</td>
  </tr>
  
   <tr>
    <td>rdf</td>
    <td>http://www.w3.org/1999/02/22-rdf-syntax-ns</td>
    <td>Resource description framework</td>
  </tr>
  
  <tr>
    <td>sp</td>
    <td>http://smartplatforms.org/terms</td>
    <td>Smart platforms root namespace</td>
  </tr>
  
  <tr>
    <td>v</td>
    <td>http://www.w3.org/2006/vcard/ns</td>
    <td>vCard namespace</td>
  </tr>
  
</table>

RDF Namespaces

#Naming conventions in SMART Ontology

We use the prefix sp to designate the http://smartplatforms.org/terms# namespace. Within this namespace, we use a simple convention to differentiate between classes and predicates

* Class names are designated by CamelCase with a capitalized first character. Examples- sp Medication, sp LabResult. 

* Predicate names are designated by camelCase with a lower-case first character. Examples- sp medication or sp valueAndUnit. 

{% highlight html %}
You may wonder why classes and predicates tend to have similar names. For example, we define a class sp:Medication as well as a predicate sp:medication. Here's why: a predicate like sp:medication is used to indicate that a clinical statement is associated with a medication; a class like sp:Medication is used to indicate that a clinical statement is a medication. For example, each sp:Fulfillment statement is associated with a sp:Medication statement via the predicate sp:medication. This makes sense when we consider a few RDF triples that expresses the basic pattern, associating a fulfillment with its medication via the sp:medication predicate:

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

{% highlight html %}
alertLevel Required: exactly 1 	[sp:Alert] 	Coded Value where code comes from AlertLevel 

belongs To Required: exactly 1 	[sp:Alert] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
notes
Optional: 0 or 1 	[sp:Alert] 	Message intended for a human recipient. [rdfs:Literal] 

http://smartplatforms.org/terms#Alert Properties

{% endhighlight  %}

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


{% highlight html %}
allergicReaction
Required: exactly 1 	[sp:Allergy] 	Coded Value where code comes from SNOMED

Reaction associated with an allergy. Code drawn from SNOMED-CT.
belongsTo
Required: exactly 1 	[sp:Allergy] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
category
Required: exactly 1 	[sp:Allergy] 	Coded Value where code comes from AllergyCategory

Category of an allergy (food, drug, other substance).
drugAllergen
Optional: 0 or 1 	[sp:Allergy] 	Coded Value where code comes from RxNorm_Ingredient

For drug allergies, an RxNorm Concept at the ingredient level (TTY='in').
drugClassAllergen
Optional: 0 or 1 	[sp:Allergy] 	Coded Value where code comes from NDFRT

Class of allergen, e.g. Sulfonamides or ACE inhibitors. RDF Code node with code drawn from

           NDF-RT: http://purl.bioontology.org/ontology/NDFRT/{NDFRT_ID}

foodAllergen
Optional: 0 or 1 	[sp:Allergy] 	Coded Value where code comes from UNII

Substance acting as a food or environmental allergen. For environmental and food substance is a UNII:
severity
Required: exactly 1 	[sp:Allergy] 	Coded Value where code comes from AllergySeverity

Severity of an allergy 

http://smartplatforms.org/terms#Allergy Properties

{% endhighlight  %}

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

{% highlight html %}
allergyExclusionName
Required: exactly 1 	[sp:AllergyExclusion] 	Coded Value where code comes from AllergyExclusion

Nature of the allergy exclusion.
belongsTo
Required: exactly 1 	[sp:AllergyExclusion] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation. 
{% endhighlight  %}

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

{% highlight html %}
belongsTo
Required: exactly 1 	[sp:Demographics] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
ethnicity
Optional: 0 or 1 	[sp:Demographics] 	[rdfs:Literal]
medicalRecordNumber
Required: 1 or more 	[sp:Demographics] 	Code
preferredLanguage
Optional: 0 or 1 	[sp:Demographics] 	[rdfs:Literal]
race
Optional: 0 or 1 	[sp:Demographics] 	[rdfs:Literal]
adr
Optional: 0 or more 	[sp:Demographics] 	Address
bday
Required: exactly 1 	[sp:Demographics] 	[rdfs:Literal]
email
Optional: 0 or more 	[sp:Demographics] 	[rdfs:Literal]
n
Required: exactly 1 	[sp:Demographics] 	Name
tel
Optional: 0 or more 	[sp:Demographics] 	Tel
gender
Required: exactly 1 	[sp:Demographics] 	A person's (administrative) gender. This should consist of the string "male" or "female". [rdfs:Literal] 
{% endhighlight  %}

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


{% highlight html %}
belongsTo
Required: exactly 1 	[sp:Encounter] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
encounterType
Required: exactly 1 	[sp:Encounter] 	Coded Value where code comes from EncounterType

Type of encounter
endDate
Optional: 0 or 1 	[sp:Encounter] 	Date when encounter ended [xsd:dateTime]
facility
Optional: 0 or 1 	[sp:Encounter] 	Organization

Facility where encounter occurred
provider
Optional: 0 or 1 	[sp:Encounter] 	Provider

Provider responsible for encounter
startDate
Required: exactly 1 	[sp:Encounter] 	Date when encounter began [xsd:dateTime] 

{% endhighlight  %}

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


{% highlight html %}
date
Required: exactly 1 	[sp:Fulfillment] 	Date on which medication was dispensed [xsd:dateTime]
belongsTo
Required: exactly 1 	[sp:Fulfillment] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
dispenseDaysSupply
Required: exactly 1 	[sp:Fulfillment] 	The number of days' supply dispensed [rdfs:Literal]
medication
Required: exactly 1 	[sp:Fulfillment] 	Medication
pbm
Optional: 0 or 1 	[sp:Fulfillment] 	The PBM providing payment for medications [rdfs:Literal]
pharmacy
Optional: 0 or 1 	[sp:Fulfillment] 	Pharmacy

The pharmacy that dispensed the medication
provider
Optional: 0 or 1 	[sp:Fulfillment] 	Provider

Clinician who prescribed the medication
quantityDispensed
Optional: 0 or 1 	[sp:Fulfillment] 	ValueAndUnit

Quantity dispensed, with units 

{% endhighlight  %}


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



{% highlight html %}
date
Required: exactly 1 	[sp:Immunization] 	ISO8601 formatted date and time when the medication was administered or offered. [xsd:dateTime]
administrationStatus
Required: exactly 1 	[sp:Immunization] 	Coded Value where code comes from ImmunizationAdministrationStatus
belongsTo
Required: exactly 1 	[sp:Immunization] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
productClass
Optional: 0 or more 	[sp:Immunization] 	Coded Value where code comes from ImmunizationClass

coded name for the product class, according to the set of codes in the CDC Vaccine Groups controlled vocabulary. For example, a class code meaning 'Rotavirus' would be assigned for a specific product such as Rotarix product.


productClass codes are drawn from the CDC's Vaccine Group vocabulary. URIs are of the form: http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#code


For example, the URI for the ROTAVIRUS code is: http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=vg#ROTAVIRUS
productName
Required: exactly 1 	[sp:Immunization] 	Coded Value where code comes from ImmunizationProduct

coded describing the product according to the set of codes in the CVX for immunizationa controlled vocabulary. CVX Code URIs should be represented as:

http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#{code}


For exampe, the code for "adenovirus, type 4" is 54, and its URI is:

http://www2a.cdc.gov/nip/IIS/IISStandards/vaccines.asp?rpt=cvx#54
refusalReason
Optional: 0 or 1 	[sp:Immunization] 	Coded Value where code comes from ImmunizationRefusalReason

If the administration status indicates this vaccination was refused, refusalReason is a CodedValue whose code is belongs to a controlled vocabulary of refusal reasons. 

{% endhighlight  %}

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




{% highlight html %}
abnormalInterpretation
Optional: 0 or 1 	[sp:LabResult] 	Coded Value where code comes from LabResultInterpretation

Abnormal interpretation status for this lab
accessionNumber
Optional: 0 or 1 	[sp:LabResult] 	External accession number for a lab result [rdfs:Literal]
belongsTo
Required: exactly 1 	[sp:LabResult] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
labName
Required: exactly 1 	[sp:LabResult] 	Coded Value where code comes from LOINC

LOINC Coded Value for result (e.g. with title='Serum Sodium' and code=http://purl.bioontology.org/ontology/LNC/2951-2
labStatus
Optional: 0 or 1 	[sp:LabResult] 	Coded Value where code comes from LabResultStatus

Workflow status of this lab value (e.g. "finalized")
narrativeResult
Optional: 0 or 1 	[sp:LabResult] 	NarrativeResult

Narrative result, if any.
notes
Optional: 0 or 1 	[sp:LabResult] 	Free-text notes about this result. [rdfs:Literal]
quantitativeResult
Optional: 0 or 1 	[sp:LabResult] 	QuantitativeResult

Qualitative result, if any
specimenCollected
Optional: 0 or 1 	[sp:LabResult] 	Attribution

Attribution specificying when specimen was collected and who was responsible for them. 

{% endhighlight  %}

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



{% highlight html %}
belongsTo
Required: exactly 1 	[sp:Medication] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
drugName
Required: exactly 1 	[sp:Medication] 	Coded Value where code comes from RxNorm_Semantic

RxNorm Concept ID for this medication. Note: the RxNorm CUI for a SMART medication should have one of the following four types: SCD (Semantic Clinical Drug), SBD (Semantic Branded Drug), GPCK (Generic Pack), BPCK (Brand Name Pack). Restricting medications to these four RxNorm types can also be expressed as "TTY in ('SCD','SBD','GPCK','BPCK')" -- and this restriction ensures that SMART medications use concepts of the appropriate specificity: concepts like "650 mg generic acetaminophen" or "20 mg brand-name Claritin". Please note that SMART medications do not include explicit structured data about pill strength, concentration, or precise ingredients. These data are available through RxNorm, including through the free RxNav REST API. Code element with code drawn from http://purl.bioontology.org/ontology/RXNORM/{rxcui}
endDate
Optional: 0 or 1 	[sp:Medication] 	

When the patient stopped taking a medication [xsd:dateTime]
frequency
Optional: 0 or 1 	[sp:Medication] 	ValueAndUnit


For a medication with a simple dosing schedule, record how often to take the medication. The frequency should be recorded as a UCUM expression. In this case we use a restricted subset of UCUM that defines the following units only: "/d" (per day), "/wk" (per week), "/mo" (per month). For example, you would express the concept of "TID" or "three times daily" as a ValueAndUnit node with quantity="3", unit="/d".
fulfillment
Optional: 0 or more 	[sp:Medication] 	Fulfillment


A single fulfillment event (that is, the medication was dispensed to the patient by a pharmacy)
instructions
Required: exactly 1 	[sp:Medication] 	

Clinician-supplied instructions from the prescription signature [rdfs:Literal]
provenance
Optional: 0 or 1 	[sp:Medication] 	Code where code comes from MedicationProvenance
quantity
Optional: 0 or 1 	[sp:Medication] 	ValueAndUnit


For a medication with a simple dosing schedule, record the amount to take with each administration. The quantity should be recorded as a UCUM expression. For some medications, the appoporiate quantity will be a volume (e.g. "5 mL" of an oral acetaminophen solution). For other medications, the appropriate quantity may be expresses in terms of tablets, puffs, or actuations: UCUM call these "non-units", and they should be written inside of curly braces to avoid confusion. For example, you would express "1 tablet" as a ValueAndUnit node with quantity="1", unit="{tablet}".
startDate
Required: exactly 1 	[sp:Medication] 	

When the patient started taking a medication [xsd:dateTime] 
{% endhighlight  %}

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

{% highlight html %}
belongsTo
Required: exactly 1 	[sp:Problem] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation.
endDate
Optional: 0 or 1 	[sp:Problem] 	Date on which problem resolved, if any. [xsd:dateTime]
notes
Optional: 0 or 1 	[sp:Problem] 	Additional notes about the problem [rdfs:Literal]
problemName
Required: exactly 1 	[sp:Problem] 	Coded Value where code comes from SNOMED

SNOMED-CT Concept for the problem
startDate
Required: exactly 1 	[sp:Problem] 	Date on which problem began [xsd:dateTime] 

{% endhighlight  %}

##SMART Statement RDF

{% highlight html %}
belongsTo
Required: exactly 1 	[sp:Statement] 	Medical Record

The medical record URI to which a clinical statement belongs. Each clinical statement points back to its medical record so that it can be treated in isolation. 

{% endhighlight  %}